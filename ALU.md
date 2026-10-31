## ALU
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;

entity ALUBATCH is
        Port ( a : in STD_LOGIC_VECTOR (3 downto 0);
               b : in STD_LOGIC_VECTOR (3 downto 0);
               sel : in STD_LOGIC_VECTOR (2 downto 0);
               y : out STD_LOGIC_VECTOR (3 downto 0));
end ALUBATCH;

architecture Behavioral of ALUBATCH is
begin
  process(a, b, sel)
  begin
    case sel is
      when "000" => y <= a + b;
      when "001" => y <= a - b;
      when "010" => y <= a AND b;
      when "011" => y <= a OR b;
      when "100" => y <= a XOR b;
      when "101" => y <= a XNOR b;
      when "111" => y <= a;
      when others => y <= "0000";
    end case;
  end process;
end Behavioral;
```
