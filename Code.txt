#include <reg51.h>          //Library file 
#include <stdio.h>
#include <string.h>

#define lcd_data P1        //Data pins connected to Port 1 of 8051 microcontroller
 
sbit rs=P2^0;              //RS pin of LCD connected to pin 0 of Port 2
sbit rw=P2^1;              //RW pin of LCD connected to pin 1 of Port 2
sbit en=P2^2;              //RS pin of LCD connected to pin 2 of Port 2
 
void lcd_init();                              //Function to initialize LCD  
void cmd(unsigned char a);                    //Function to send command instructions to LCD
void dat(unsigned char b);                    //Function to send display data to LCD
void show(unsigned char *s);                  //Function to display data
void lcd_delay();                             //Function to create delay on LCD
void delay1(unsigned int time);               //Function to create delay
unsigned char rx();                           //Function to store received input
char time[5];
char namegps[7], name1gps[7] = "GPRMC,",gpsdat[63];
char msggps , checkgps;
int h;
int x,y,z;
char a,b,c;
unsigned char f;

void main()                                 //Main function
{
    TMOD = 0x20;                            //Timer 1 in mode 2    (auto reload mode)        
    TH1 = TL1=0xfd;                         //baud rate=9600
    SCON = 0x50;                            //Serial mode 1, 8 bit data, 1 stop bit and 1 start bit
    TR1 = 1;                                //Start timer 1
    lcd_init();
    cmd(0x80);                              //Force cursor to beginning of first line
	show("***WELCOME TO***");
	cmd(0xc0);                              //Force cursor to beginning of second line
	show("***GPS SYSTEM***");
	 delay1(500);                           //Delay of 500ms
	 cmd(0x01);                             //Clear display screen
	 cmd(0x80);                             
	 show("WAITING FOR");
	 cmd(0xc0);
	show("GPS CONNECTION"); 
	 delay1(1000);                          //Delay of 1000ms

    while(1)                                //If the above conditions are satisfied proceed

  {
   AGAIN:
   loop: 
	while(RI==0);                          //Receiver Interrupt
    	 RI=0;
		 if(SBUF!='$')                      //Starting character
	     {
		  goto loop;
	     }
		  for(f=0;f<=5;f++)                  //To receive 6 successive characters
		  {
                namegps[f]=rx();           //Store received characters in the function rx()
          }
            checkgps=strcmp(namegps,name1gps);     //String comparison (checkgps = 0; strings are equal)   
		   if(checkgps==0) 
		   {
                for(f=0;f<=62;f++)                //Get next 63 characters
				{
                    gpsdat[f]=rx();     
                }
           }
				if(gpsdat[10]=='A')               //If character 'A' is received it means that GPS is activated
				{
				cmd(0x01);
				cmd(0x80);
				show("GPS CONNECTED");
				cmd(0xc0);
				show("****************");
				delay1(200);
				}
				else
				{
				goto AGAIN; 
				}
				cmd(0x01);
				cmd(0x80);
				show("LAT:");                    //Displaying Latitude values
				cmd(0x84);                       //Cursor line 1 position 4
                for(h=12;h<14;h++)               //To get Latitude value in degree
				{
                    dat(gpsdat[h]);
                }
                dat(223);
                for(h=14;h<=15;h++)             //To get Latitude value in minute
				{
                    dat(gpsdat[h]);
                }
				dat(39);
                for(h=17;h<=18;h++)             //To get Latitude value in seconds
			   {
                    dat(gpsdat[h]);
                }
				dat(34);
                dat('N');
				cmd(0xc0);
				show("LON:");                   //To display Longitude values
                cmd(0xc4);                      //Jump to second line, position 4
                for(h=26;h<28;h++)              //To get Longitude value in degree
				{
                    dat(gpsdat[h]);
                }
                dat(223);
                for(h=28;h<30;h++)              //To get Longitude value in minutes
				{
                    dat(gpsdat[h]);
                }
				dat(39);
                for(h=31;h<=32;h++)              //To get Longitude value in seconds
				{
                    dat(gpsdat[h]);
                }
				dat(34);
                dat('E');
			   delay1(2000);
			   cmd(0x01);
			   cmd(0x80);
			   show("      DATE      ");          //To display date
			   cmd(0xc4);
			   for(h=46;h<=47;h++)
			   {
			   dat(gpsdat[h]);
			   }
			   	dat('/');
				for(h=48;h<=49;h++)
			   {
			   dat(gpsdat[h]);
			   }
			   	dat('/');
				for(h=50;h<=51;h++)
			   {
			   dat(gpsdat[h]);
			   }
				delay1(1000);
				cmd(0x01);
			   cmd(0x80);
			   show("      TIME      ");           //To display time 
			    cmd(0xc6);
			    //To display time in UTC format 
			    dat(gpsdat[0]);
			    dat(gpsdat[1]);
			    dat(':');
			    dat(gpsdat[2]);
			    dat(gpsdat[3]);
			    dat(':');
			    dat(gpsdat[4]);
			    dat(gpsdat[5]);
			    delay1(2000);
               
			     /* Program to convert time from UTC format to IST format
			        x = gpsdat[2]-48;
				y=gpsdat[1]-48;
				z=gpsdat[0]-48;
				x=x+3;
				if(x>=6)
				{
				 	x=x-6;
					y=y+1;
				}
				y=y+5;
				if(y>=10)
				{
				 	y=y%10;
					z=z+1;
				}
				if(z==2 & y>4)
				{
				  y=y-4;
				  z=0;
				}
				if(z==1 & y>2 )
				{
				 z=0;
				 y=y-2;
				}
				if(z==2 & y>=0)
				{
				 if(y==0)
				 {
				  z=0;
				  y=8;
				 }
				 else if(y==1)
				 {
				  z=0;
				  y=9;
				 }
				 else if(y==2)
				 {
				  z=1;
				  y=0;
				 }
				 else if(y==3)
				 {
				  z=1;
				  y=1;
				 }
				 else if(y==4)
				 {
				  z=0;
				  y=0;
				 }
				}
				a = x+48;
				b = y+48;
				c = z+48;
				

				cmd(0xc6);
				dat(c);
				dat(b);
				dat(':') ;
				dat(a);
				dat(gpsdat[3]);
				delay1(2000);
				*/
	}	
}


unsigned char rx()
{
    while(RI==0);
    RI=0;
    return SBUF;
}

void lcd_init()
{
    cmd(0x38);            //2 lines and 5x7 matrix
    cmd(0x0e);            //Display ON, cursor blinking
    cmd(0x01);            //Clear display screen
    cmd(0x06);            //Increment cursor (shift cursor to right)
    cmd(0x0c);            //Display ON, cursor OFF
    cmd(0x80);            //Force cursor to beginning of first line
}
 
void cmd(unsigned char a)
{
    lcd_data=a;
    rs=0;            //Command register
    rw=0;            //Write operation
    en=1;            //Enable Pulse
    lcd_delay();
    en=0;
}
 
void dat(unsigned char b)
{
    lcd_data=b;       
    rs=1;            //Data register
    rw=0;            //Write operation
    en=1;            //Enable pulse
    lcd_delay();
    en=0;
}
 
void show(unsigned char *s)         //Display string
{
    while(*s)                     
	{
        dat(*s++);             
    }
}
void delay1(unsigned int time)
{
     unsigned int i;
     unsigned int j;
       for(i=0;i<time;i++)
       for(j=0;j<1275;j++);
}
/* DELAY CALCULATIONS
f=11.0592 MHz
The timer works with a clock frequency of 1/12 of the XTAL frequency.
f/12 = 11.0592 MHz / 12 = 921.6 kHz
T =1/921.6kHz = 1.085us. This is the one machine cycle time. 
1000us = 1ms
so that 1.085us X 1275 = 1.383 ms (appx 1ms delay)
*/
void lcd_delay()
{
    unsigned int lcd_delay;
    for(lcd_delay=0; lcd_delay<=6000; lcd_delay++);            //Delay of 6ms
}
