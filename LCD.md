## LCD Code
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity lcd is
    Port (
        clk_12Mhz : in  STD_LOGIC;
        lcd_rs    : out STD_LOGIC;
        lcd_en    : out STD_LOGIC;
        lcd_data  : out STD_LOGIC_VECTOR(7 downto 0);
        rst       : in  STD_LOGIC
    );
end lcd;

architecture Behavioral of lcd is

    signal div : STD_LOGIC_VECTOR(19 downto 0)
                := (others => '0');
    signal clk_t1 : STD_LOGIC := '0';

    type state is (
        init0, init1, init2, init3, set_ddram,
        init0_en, init1_en, init2_en, init3_en,
        ddram_en, char_setup, char_en, char_done, done
    );

    signal ps : state := init0;

    constant MESSAGE : string := "FIFO READY";

    signal char_index : natural range 0 to MESSAGE'length := 0;

    signal lcd_rs_s   : STD_LOGIC := '0';
    signal lcd_en_s   : STD_LOGIC := '0';
    signal dataout_s  : STD_LOGIC_VECTOR(7 downto 0)
                       := (others => '0');

begin

    lcd_rs   <= lcd_rs_s;
    lcd_en   <= lcd_en_s;
    lcd_data <= dataout_s;

    -- Clock divider
    process(clk_12Mhz, rst)
    begin
        if rst = '1' then
            div <= (others => '0');
        elsif rising_edge(clk_12Mhz) then
            div <= std_logic_vector(unsigned(div) + 1);
        end if;
    end process;

    clk_t1 <= div(19);

    -- LCD controller FSM
    process(clk_t1, rst)
    begin
        if rst = '1' then
            ps         <= init0;
            lcd_rs_s   <= '0';
            lcd_en_s   <= '0';
            dataout_s  <= (others => '0');
            char_index <= 0;

        elsif rising_edge(clk_t1) then

            -- Default: enable low
            lcd_en_s <= '0';

            case ps is

                -- Function set: 8-bit, 2-line mode
                when init0 =>
                    lcd_rs_s  <= '0';
                    dataout_s <= x"38";
                    ps <= init0_en;

                when init0_en =>
                    lcd_en_s <= '1';
                    ps <= init1;

                -- Display ON, cursor OFF
                when init1 =>
                    lcd_rs_s  <= '0';
                    dataout_s <= x"0C";
                    ps <= init1_en;

                when init1_en =>
                    lcd_en_s <= '1';
                    ps <= init2;

                -- Clear display
                when init2 =>
                    lcd_rs_s  <= '0';
                    dataout_s <= x"01";
                    ps <= init2_en;

                when init2_en =>
                    lcd_en_s <= '1';
                    ps <= init3;

                -- Entry mode: increment cursor
                when init3 =>
                    lcd_rs_s  <= '0';
                    dataout_s <= x"06";
                    ps <= init3_en;

                when init3_en =>
                    lcd_en_s <= '1';
                    ps <= set_ddram;

                -- Start at first position of line 1
                when set_ddram =>
                    lcd_rs_s  <= '0';
                    dataout_s <= x"80";
                    ps <= ddram_en;

                when ddram_en =>
                    lcd_en_s <= '1';
                    char_index <= 0;
                    ps <= char_setup;

                -- Send message characters
                when char_setup =>
                    if char_index < MESSAGE'length then
                        lcd_rs_s <= '1';
                        dataout_s <= std_logic_vector(
                            to_unsigned(
                                character'pos(
                                    MESSAGE(char_index + 1)
                                ), 8
                            )
                        );
                        ps <= char_en;
                    else
                        ps <= done;
                    end if;

                when char_en =>
                    lcd_en_s <= '1';
                    ps <= char_done;

                when char_done =>
                    lcd_en_s <= '0';
                    if char_index < MESSAGE'length then
                        char_index <= char_index + 1;
                    end if;
                    ps <= char_setup;

                when done =>
                    lcd_en_s <= '0';
                    ps <= done;

                when others =>
                    ps <= init0;

            end case;
        end if;
    end process;

end Behavioral;
```
