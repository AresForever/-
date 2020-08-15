# -
可以测量多种波形的频率、电压峰峰值
#include <reg52.h>

typedef unsigned char u8;
typedef unsigned short u16;

//ADC0808µÄÒý½ÅÅäÖÃ
sbit ST=P3^0;
sbit EOC=P3^1;
sbit OE=P3^3;
sbit ALE=P3^4;
sbit addA=P3^5;
sbit addB=P3^6;
sbit addC=P3^7;

//LM016LÒº¾§ÏÔÊ¾Òý½ÅÅäÖÃ
sbit RS=P1^0;
sbit RW=P1^1;
sbit EN=P1^2;

//×ª»»°´¼ü
sbit CS=P1^3;

void init();
void lcd_init();
void break_init();
void set_val(u8 x,u8 y,u8 z);
void delay(u8 x);
void write_com(u8 date);
void write_data(u8 date);
void deal_vol();
void deal_AU();
void deal_freq();
void clear_array(u8 *s);
void get_KeyVal();
void get_ascii(float x);
void get_freq();
u8 get_vol(u8 x,u8 y,u8 z);

u8 name[]={"U:"};
u8 name1[]={"Au:"};
u8 name2[5]={0};
u8 digit[]={"0123456789"};
u8 frequence[]={"Freq:"};
u8 s=0;
u16 time_count,count,sum;

void lcd_init()
{
	write_com(0x38);
	write_com(0x0f);
	write_com(0x06);
	write_com(0x01);
	delay(50);
}

void break_init()  //Íâ²¿ºÍÊ±ÖÖÖÐ¶Ï³õÊ¼»¯£¬²âÆµÂÊ
{
	count=0;
	time_count=0;
  sum=0;
	TMOD=0x02;
	TH0=0x06;
	TL0=0x06; 
	IT0=1;
	EA=1;
	ET0=1;
	EX0=1;
	TR0=1; 
}

//³õÊ¼»¯
void init() 
{
	lcd_init();
	break_init();
	ALE=0;
	ST=0;
	OE=0;
}

//ÉèÖÃÄ£ÄâÍ¨µÀ
void set_val(u8 x,u8 y,u8 z)
{
	addA=x;
	addB=y;
	addC=z;
  ALE=0;
	delay(5);
	ALE=1;
	delay(5);
	ALE=0;
}

void delay(u8 x)
{
	while(x--);
}

void write_com(u8 date)
{
	RS=0;
	RW=0;
	EN=0;
	delay(50);
	EN=1;
	delay(50);
	P0=date;
	delay(3);
	EN=0;
}

void write_data(u8 date)
{
	RS=1;
	RW=0;
	EN=0;
	delay(50);
	EN=1;
	delay(50);
	P0=date;
	delay(50);
	EN=0;
}

u8 get_vol(u8 x,u8 y,u8 z) //µÃµ½µçÑ¹Öµ
{
	u8 date;
	set_val(x,y,z); 
	OE=0;
	ST=0;
	delay(5);
	ST=1;
	delay(5);
	ST=0;
	delay(5);
	while(EOC==0);
	OE=1;
	delay(1);
	date=P2;
	OE=0;
	return date;
}

void clear_array(u8 *s)
{
	u8 i;
	for(i=0;i<5;i++)
		s[i]=0;
}

void get_ascii(float x)
{
	u8 i=4;
	int s=(int)(x*100);
	while(s!=0)
	{
		name2[i]=s%10;
		s/=10;
		i--;
		if(i<0)
			break;
	}
}

void get_freq()
{
	u8 i=4;
	u16 x=sum;
	while(x!=0)
	{
		name2[i]=x%10;
		x/=10;
		i--;
		if(i<0)
			break;
	}
}

void deal_vol()
{
	int i;
	write_com(0x80);
	for(i=0;i<2;i++)
	{
		write_data(name[i]);
	}
	for(i=0;i<3;i++)
	{
		write_data(digit[name2[i]]);
	}
	write_data('.');
	for(i=3;i<5;i++)
	{
		write_data(digit[name2[i]]);
	}
	write_data('V');
}

void deal_AU()
{
	u8 i;
	write_com(0x80+0x40);
	for(i=0;i<3;i++)
	{
		write_data(name1[i]);
	}
	for(i=0;i<3;i++)
	{
		write_data(digit[name2[i]]);
	}
	write_data('.');
	for(i=3;i<5;i++)
		write_data(digit[name2[i]]);
}

void deal_freq()
{
	u8 i;
	write_com(0x80);
	for(i=0;i<5;i++)
		write_data(frequence[i]);
	for(i=0;i<5;i++)
		write_data(digit[name2[i]]);
	write_data('H');
	write_data('Z');
}

void get_KeyVal()
{
	if(CS!=1)
	{
		delay(1000);
		if(CS==0)
		{
			s++;
			write_com(0x01);
		}
	}
}

	
void main()
{
	u8 data1=0,data2=0,data3=0;
	float vol1=0,vol2=0,vol3=0,Au=0;
	init();
	while(1)
	{
		get_KeyVal();
		if(s%2==0)
		{
			clear_array(name2);
			data1=get_vol(0,0,0);
			vol1=(float)(data1*5*3/255);
			get_ascii(vol1);
			deal_vol();
			clear_array(name2);
			data2=get_vol(0,1,0);
			vol2=(float)(data2*5*3/255);
			data3=get_vol(1,0,1);
			vol3=(float)(data3*5*3/255);
			Au=(float)(vol3/vol2);
			get_ascii(Au);
			deal_AU();
		}
		else
		{
			clear_array(name2);
			get_freq();
			deal_freq();
		}
	}
}

void exter0() interrupt 0
{
	count++;
}

void timer0() interrupt 1
{
	time_count++;
	if(time_count==4000)
	{
		sum=count;
		time_count=0;
		count=0;
	}	
}
