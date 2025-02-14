# UART-Module-with-VHDL
Transmitter : 
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity T_UART is
    generic(
        CLK_FRQ  : integer := 100000;  
        BAUD_RATE: integer := 9600;     
        STOP_BIT : integer := 1             
    );
    port(
        CLK1      : in std_logic;            
        Tin_Data  : in std_logic_vector(7 downto 0); 
        tx_out    : out std_logic;          
        Tx_active : out std_logic;
        Tx_busy   : out std_logic            
    );
end T_UART;

architecture HAS of T_UART is
    constant BIT_TIMER : integer := CLK_FRQ / BAUD_RATE; 
    signal T_active : std_logic; 
    type Tx_STATE is (SLEEP, START, Tx_DATA, STOP);
    signal state : Tx_STATE := SLEEP; 
    signal S_DATA : std_logic_vector(7 downto 0);
    signal t_out  : std_logic;   
    signal time_counter : integer := 0; 
    signal bit_counter  : integer := 0; 
begin
 process(CLK1)
  begin
  T_active <= Tin_DATA(0) or Tin_DATA(1) or Tin_DATA(2) or Tin_DATA(3) or Tin_DATA(4) or Tin_DATA(5) or Tin_DATA(6) or Tin_DATA(7);
  if rising_edge(CLK1) then
  case state is
  when SLEEP => t_out <= '1';
                time_counter <= 0; 
   if T_active = '1' then
        state <= START; 
   end if;          
  when START => t_out <= '0';
     if (time_counter = BIT_TIMER - 1) then
        S_DATA <= Tin_DATA;
        time_counter <= 0; 
        state <= Tx_DATA;
     else
        time_counter <= time_counter + 1;
     end if;
   when Tx_DATA => t_out <= S_DATA(bit_counter);
     if (time_counter = BIT_TIMER - 1) then
        time_counter <= 0;  
        bit_counter <= bit_counter + 1;
       if bit_counter = 7 then
        bit_counter <= 0; 
        state <= STOP;
       end if;
      else
        time_counter <= time_counter + 1;
      end if;
    when STOP => t_out <= '1';
     if (time_counter = STOP_BIT * (BIT_TIMER - 1)) then
        time_counter <= 0;
        state <= SLEEP;
     else
        time_counter <= time_counter + 1;
     end if;
     end case;
     end if;
   end process;
   tx_out <= t_out;
   Tx_active <= T_active;
   Tx_busy <= '1' when (state /= SLEEP) else '0';
end HAS;
