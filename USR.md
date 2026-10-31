## Universal Shift Register
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;

entity shiftregister is
    Port ( clk     : in  STD_LOGIC;
           reset   : in  STD_LOGIC;
           datain  : in  STD_LOGIC_VECTOR (3 downto 0);
           sel     : in  STD_LOGIC_VECTOR (1 downto 0);
           sl_in   : in  STD_LOGIC;
           sr_in   : in  STD_LOGIC;
           dataout : out STD_LOGIC_VECTOR (3 downto 0));
end shiftregister;

architecture Behavioral of shiftregister is
    signal TEMP : STD_LOGIC_VECTOR(3 downto 0) := "0000";
begin

    process(reset, clk)
    begin
        if reset = '1' then
            TEMP <= "0000";

        elsif rising_edge(clk) then
            case sel is
                when "11" =>                           -- parallel load
                    TEMP <= datain;
                when "00" =>                           -- hold
                    null;
                when "01" =>                           -- left shift
                    TEMP <= TEMP(2 downto 0) & sl_in;
                when "10" =>                           -- right shift
                    TEMP <= sr_in & TEMP(3 downto 1);
                when others =>
                    null;
            end case;
        end if;
    end process;

    dataout <= TEMP;

end Behavioral;
```
