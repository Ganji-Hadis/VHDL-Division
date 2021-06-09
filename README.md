# VHDL-Division
Division by Shift & Sub


    library IEEE;
    use IEEE.STD_LOGIC_1164.ALL;
     use IEEE.numeric_std.ALL;
    --USE ieee.std_logic_arith.all;

     --------------------------------------------------------------------------
    entity div is
  	port( AQ : in std_logic_vector(7 downto 0);
	  		B : in std_logic_vector(3 downto 0);
        remainder : out std_logic_vector(3 downto 0);
	  		quotient :out std_logic_vector(3 downto 0);
		  	DVF : out std_logic);
    end div;
    --------------------------------------------------------------------------
    architecture Behavioral of div is
	--signal A : std_logic_vector(3 downto 0);
	--signal Q : std_logic_vector(3 downto 0);
	--signal EA : std_logic_vector(4 downto 0) ;
	--signal EAQ : std_logic_vector(8 downto 0) ;
    begin

    process(AQ , B)

	variable A : std_logic_vector(3 downto 0);
	variable Q : std_logic_vector(3 downto 0);
	variable EA : std_logic_vector(4 downto 0) ;
	variable EAQ : std_logic_vector(8 downto 0) ;
	variable one : std_logic_vector(3 downto 0) ;
	variable temp : std_logic_vector(7 downto 0);
	
	begin 
	
		A := AQ(7 downto 4);
		Q(3 downto 0) := AQ(3 downto 0);
		--Q(3) <= A(3) xor B(3);
		EA := '0' & A ;
		--EAQ <= '0' & A & Q ;
		one := "0001" ;
		
		  
		EA := std_logic_vector(unsigned('0' & A) + unsigned('0' & not(B)) + unsigned('0' & one)) ;  -- B' + 1 + A
		
		if(EA(4) = '0') then
		
			DVF <= '0';
			EAQ := '0' & A & Q ;
			for i in 0 to 3 loop
			
				temp := EAQ(7 downto 0);
				EAQ :=  temp & '0' ;  -- shift
				--Q <= EAQ(3 downto 1) & '0' ;
		
					if( EAQ(8) = '1' ) then  -- E is negative?
						EAQ(7 downto 4) := std_logic_vector(unsigned(not(B)) + unsigned(one) + unsigned(EAQ(7 downto 4))) ;  -- B' + 1 + A
						EAQ(0) := '1';
						
					else 
						EAQ(8 downto 4) := std_logic_vector(unsigned('0' & not(B)) + unsigned('0' & one) + unsigned('0' & EAQ(7 downto 4))) ;  -- B' + 1 + A
						
					  	if(EAQ(8) = '1') then
						  	EAQ(0) := '1';
						 
					  	else
				  			EAQ(8 downto 4) := std_logic_vector('0' & unsigned(B)+ unsigned('0' & EAQ(7 downto 4))) ;
				  		end if;
						
	  				end if;
	  		end loop;
			
		  		remainder <= EAQ(7 downto 4);
		  		quotient <= EAQ(3 downto 0) ;
				
  		else
		  	EA := '0' & std_logic_vector(unsigned(EA(3 downto 0)) + unsigned(B)) ;
		  	DVF <= '1';
		  	remainder <= "0000" ;
		  	quotient <= "0000" ;	
			
		  end if;	
			
	  end process;
    end Behavioral;




#TestBench

    LIBRARY ieee;
    USE ieee.std_logic_1164.ALL;
 
    -- Uncomment the following library declaration if using
    -- arithmetic functions with Signed or Unsigned values
    --USE ieee.numeric_std.ALL;
 
    ENTITY divTest IS
    END divTest;
 
    ARCHITECTURE behavior OF divTest IS 
 
    -- Component Declaration for the Unit Under Test (UUT)
 
    COMPONENT div
    PORT(
         AQ : IN  std_logic_vector(7 downto 0);
         B : IN  std_logic_vector(3 downto 0);
         remainder : OUT  std_logic_vector(3 downto 0);
         quotient : OUT  std_logic_vector(3 downto 0);
			DVF : out std_logic
        );
    END COMPONENT;
    

      --Inputs
      signal AQ : std_logic_vector(7 downto 0) := (others => '0');
      signal B : std_logic_vector(3 downto 0) := (others => '0');

 	--Outputs
     signal remainder : std_logic_vector(3 downto 0);
     signal quotient : std_logic_vector(3 downto 0);
  	SIGNAL DVF : std_logic;
  
     -- No clocks detected in port list. Replace <clock> below with 
    -- appropriate port name 
 
       --constant <clock>_period : time := 10 ns;
 
      BEGIN
 
	-- Instantiate the Unit Under Test (UUT)
      uut: div PORT MAP (
           AQ => AQ,
          B => B,
          remainder => remainder,
          quotient => quotient,
			 DVF => DVF
        );

       -- Clock process definitions
       --<clock>_process :process
      -- begin
	    --	<clock> <= '0';
	    --	wait for <clock>_period/2;
  	--	<clock> <= '1';
	  --	wait for <clock>_period/2;
    -- end process;
 

     -- Stimulus process
    stim_proc: process
     begin		
      -- hold reset state for 100 ns.
     --  wait for 100 ns;	

     -- wait for <clock>_period*10;

      -- insert stimulus here 

		AQ <= "00110000" ;  -- 48
		B <= "0110" ;  --6
		--Q = 8 
		--R = 0
		--DVF = 0
		wait for 10 ns;
		
		AQ <= "00010010";  -- 18
		B <= "0100" ;  --4
		--Q = 4 
		--R = 2
		--DVF = 0
		wait for 10 ns;
		
		AQ <= "01000100";  -- 68
		B <= "0010" ;  --2
		--Q = 0 
		--R = 0
		--DVF = 1
		
		
       wait;
      end process;

    END;

