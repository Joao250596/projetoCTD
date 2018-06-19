# projetoCTD
-------------------------------------PROJETO CTD-----------------------------------------

////////////////////////////////////////////////////////////////////////////////////////

------------Memória (RAM) para códigos do jogo---------------------
library ieee;
use ieee.std_logic_1164.all;

entity ROM is
  port ( address : in std_logic_vector(3 downto 0);
         data : out std_logic_vector(9 downto 0) );
end entity;

architecture Rom_Arch of ROM is
  type memory is array (00 to 15) of std_logic_vector(9 downto 0);
  constant my_Rom : memory := (
	00 => "0101001100",
	01 => "1010010100",
  02 => "1101001010",
	03 => "1010100100",
	04 => "1110001000",
	05 => "0010010101",
	06 => "1010110000",
	07 => "1010010100",
	08 => "1000110010",
	09 => "1010010100",
	10 => "1010001100",
	11 => "1000000111",
	12 => "0010110100",
	13 => "1010010100",
	14 => "1010101000",
	15 => "0010010101",);
begin
   process (address)
   begin
     case address is
       when "0000" => data <= my_rom(00);
       when "0001" => data <= my_rom(01);
       when "0010" => data <= my_rom(02);
       when "0011" => data <= my_rom(03);
       when "0100" => data <= my_rom(04);
       when "0101" => data <= my_rom(05);
       when "0110" => data <= my_rom(06);
       when "0111" => data <= my_rom(07);
       when "1000" => data <= my_rom(08);
       when "1001" => data <= my_rom(09);
		 when "1010" => data <= my_rom(10);
		 when "1011" => data <= my_rom(11);
		 when "1100" => data <= my_rom(12);
		 when "1101" => data <= my_rom(13);
		 when "1110" => data <= my_rom(14);
		 when "1111" => data <= my_rom(15);
       when others => data <= "0000000000";
       end case;
  end process;
end architecture Rom_Arch;

/////////////////////////////////////////////////////////////////////////////////////////////

----------------------------Topo do projeto---------------------------

library IEEE;
use ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;

entity Projeto is
port(
selecInicio: in std_logic_vector (3 downto 0);
selecJogada: in std_logic_vector (9 downto 0);
enter: in std_logic;
reset: in std_logic;
CLOCK_50: in std_logic
);
end;


///////////////////////////////////////////////////////////////////////////////////////

---------------------------Bloco de Controle--------------------------

library IEEE;
use ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;

entity Controle is
port (
enter, sw_error, end_game, end_time, end_round, reset: in std_logic;
clock: in std_logic;
R1 , R2, E1, E2, E3, E4, E5: out std_logic
);
end;

architecture Bloco of Controle is
type STATES is (Start,Setup,Play,Check,Next_Round,Result);
signal EA, PE: STATES;
begin

process (clock,reset)
begin
   if reset = '0' then 
	EA <= Start;
   elsif clock'event and clock= '1' then 
	EA <= PE;
end if;
end process;

process (EA, enter,end_time,end_round,sw_error,end_game)
begin
			case EA is
      		when Start => 
				R1 <= '1'; 
				R2 <= '1'; 
				E1 <= '0'; 
				E2 <= '0'; 
				E3 <= '0'; 
				E4 <= '0';
				E5 <= '0'; 
          	if enter = '0' then 
					PE <= Setup;
          	else 
					PE <= Start;
          	end if;
          
          when Setup => 
			 R1 <= '0'; 
			 R2 <= '0'; 
			 E1 <= '0'; 
			 E2 <= '1'; 
			 E3 <= '0'; 
			 E4 <= '0'; 
			 E5 <= '0';
          if enter = '1' then 
				PE <= Setup;
          else 
				PE <= Play;
          end if;
   
          when Play => 
			 R1 <= '0'; 
			 R2 <= '0'; 
			 E1 <= '1'; 
			 E2 <= '0'; 
			 E3 <= '0'; 
			 E4 <= '0'; 
			 E5 <= '0';
         if (enter ='1'  and end_time ='0') then 
				PE <= Play;
         elsif (enter ='0'  and end_time ='0') then 
				PE <= Check;
         else 
				PE <= Result;
         end if;
                        
          when Check => 
			 R1 <= '0'; 
			 R2 <= '0'; 
			 E1 <= '0'; 
			 E2 <= '0'; 
			 E3 <= '1'; 
			 E4 <= '0'; 
			 E5 <= '0';
          if (end_round= '0' and sw_error = '0' and end_game = '0') then 
				PE <= Next_Round;
         else 
				PE <= Result;
         end if;
                        
   				when Next_Round => 
					R1 <= '1'; 
					R2 <= '0'; 
					E1 <= '0'; 
					E2 <= '0'; 
					E3 <= '0'; 
					E4 <= '1'; 
					E5 <= '0';
          		if enter= '1' then 
						PE <= Next_Round;
          		else 
						PE <= Play;
          	   end if;
                        
   				when Result => 
					R1 <= '0'; 
					R2 <= '0'; 
					E1 <= '0'; 
					E2 <= '0'; 
					E3 <= '0'; 
					E4 <= '1'; 
					E5 <= '1';
					if enter= '1' then 
						PE <= Result;
          		else 
						PE <= Start;
          		end if;
		end case;
	end process;
end Bloco;

///////////////////////////////////////////////////////////////////////////////////////

-------------------------------------Bloco Datapath------------------------------------

library IEEE;
use ieee.std_logic_1164.all;
use ieee.std_logic_unsigned.all;
use ieee.std_logic_arith.all;
 
entity of Datapath is
port (
     R1, E1, E5, clock : in std_logic;
     SW: in std_logic_vector (9 downto 0);
     sw_error, end_game, end_time, end_round: out std_logic;
     S0, S1, S2, S3, S4, S5: out std_logic_vector (6 downto 0)
);
end;

architecture arqdata of Datapath is
signal A0, C1, C2: std_logic_vector (9 downto 0); 
signal pontos: std_logic_vector (4 downto 0); 
signal t, N, N5, N10, LR, LT: std_logic_vector (3 downto 0);
signal N4, BB: std_logic_vector (2 downto 0);
signal C3, RB, ET, RT, ER, RT, RN: std_logic ;

component ROM is
  port ( address : in std_logic_vector(3 downto 0);
         data : out std_logic_vector(9 downto 0) 
);
end component;

component Comparador10bits is
port (
)
end component;

component Comparador4bits is 
port (
);
end component;

component Comparador3bits is
port (

);
end component;

component Registrador10bits is
port (
CLK: in std_logic;
entrada: in std_logic_vector (9 downto 0);
saida: out std_logic_vector (9 downto 0);
);
end component; 

component decod7seg is
port ( 
C: in std_logic_vector(3 downto 0);
H: out std_logic_vector(6 downto 0)
);
end component; 

begin

SETUP: port map ROM (SW(9 downto 6),A0);


REG1: port map --portmap(s) do(s) registrado(es)
REG2: port map --portmap(s) do(s) registrado(es)
REG3: port map --portmap(s) do(s) registrado(es)

COMPARADOR1: portmap
COMPARADOR2: portmap
COMPARADOR3: portmap

DISPLAY1: port map decod7seg (--parte menos significativa da pontuação do jogador, S0);
DISPLAY2: port map decod7seg (--parte mais significativa da pontuação do jogador, S1);
DISPLAY3: port map decod7seg (t, S2);
 --decodificador que mostra a contagem em segundos do tempoDISPLAY4: port map decod7seg (LT, S3); --decodificador que mostra a letra ""t" de tempo
DISPLAY5: port map decod7seg (N, S4); --decodificador que mostra o número de rodadas
DISPLAY6: port map decod7seg (LR, S5) --decodificador que mostra a letra "r" de rodadas

end arqdata;

////////////////////////////////////////////////////////////////////////////////////



---------------------------------Registrador de 10 bits---------------------------------------

library IEEE;
use ieee.std_logic_1164.all;

entity of Registrador10bits is
port(
CLK: in std_logic;
entrada: in std_logic_vector (9 downto 0);
saida: out std_logic_vector (9 downto 0)
);
end;

architecture arq of Registrador10bits is
begin
begin
 process(CLK, entrada)
 begin
 if (CLK'event and CLK = '1') then
 saida <= entrada;
 end if;
 end process;
end arq;

////////////////////////////////////////////////////////////////////////////////////

******como fazer comparadores de igualdade******

---------------------------------Comparador 10bits--------------------------------------

library IEEE;
use ieee.std_logic_1164.all;

entity of Comparador10bits is
port (
sinalA, sinalB : in std_logic_vector (9 downto 0);
resultado: out std_logic
);
end;

achitecture logica of Comparador10bits is
signal ramo1, ramo2, ramo3, ramo4, ramo5, ramo6, ramo7, ramo8, ramo9, ramo10???? 	
  begin
	
resultado <= sinalA NXOR   

////////////////////////////////////////////////////////////////////////////////////

----------------------------Contador tempo------------------------------------------

library IEEE;
use ieee.std_logic_1164.all;

entity of ContadorTempo is
port (
enable, reset, clk: in std_logic;
t: out std_logic_vector (3 downto 0);
);
end;

achitecture ContArq of ContadorTempo is
