## FIFO
``` vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity fifo1 is
    Port (
        clk     : in  STD_LOGIC;
        reset   : in  STD_LOGIC;
        WriteEn : in  STD_LOGIC;
        DataIn  : in  STD_LOGIC_VECTOR(1 downto 0);
        ReadEn  : in  STD_LOGIC;
        DataOut : out STD_LOGIC_VECTOR(1 downto 0);
        Empty   : out STD_LOGIC;
        Full    : out STD_LOGIC
    );
end fifo1;

architecture Behavioral of fifo1 is

    constant FIFO_DEPTH : natural := 4;

    type FIFO_Memory is array (0 to FIFO_DEPTH-1)
        of STD_LOGIC_VECTOR(1 downto 0);

    signal Memory : FIFO_Memory :=
        (others => (others => '0'));

    signal Head : natural range 0 to FIFO_DEPTH-1 := 0;
    signal Tail : natural range 0 to FIFO_DEPTH-1 := 0;
    signal Count : natural range 0 to FIFO_DEPTH := 0;

    signal dataOutReg : STD_LOGIC_VECTOR(1 downto 0)
        := "00";

begin

    DataOut <= dataOutReg;
    Empty <= '1' when Count = 0 else '0';
    Full  <= '1' when Count = FIFO_DEPTH else '0';

    fifo_proc : process(clk, reset)
    begin
        if reset = '1' then
            Head       <= 0;
            Tail       <= 0;
            Count      <= 0;
            dataOutReg <= "00";

        elsif rising_edge(clk) then

            if WriteEn = '1' and ReadEn = '0'
               and Count < FIFO_DEPTH then

                Memory(Tail) <= DataIn;

                if Tail = FIFO_DEPTH-1 then
                    Tail <= 0;
                else
                    Tail <= Tail + 1;
                end if;

                Count <= Count + 1;

            elsif ReadEn = '1' and WriteEn = '0'
               and Count > 0 then

                dataOutReg <= Memory(Head);

                if Head = FIFO_DEPTH-1 then
                    Head <= 0;
                else
                    Head <= Head + 1;
                end if;

                Count <= Count - 1;

            end if;

        end if;
    end process;
end Behavioral;
```
