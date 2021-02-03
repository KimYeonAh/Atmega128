#include <mega128.h>
#include <delay.h>

#define TRIGGER     PORTD.0
                                              
flash unsigned char seg_pat[10]= {0x3f, 0x06, 0x5b, 0x4f, 0x66, 0x6d, 0x7d, 0x07, 0x7f, 0x6f};
flash unsigned char seg_on[4] = {0x08, 0x04, 0x02, 0x01};

unsigned char echo_st = 0;
unsigned char dsp_no = 0;           // 세그먼트 표시 번호
unsigned int  dist = 0, cnt = 0;
unsigned int  N1000, N100, N10, N1;
unsigned char connect = 0;


void  Distance_out(void);            // 거리 출력 자리값 추출

void main(void)
{
    int i;      // 반복문
    //RC서보모터
    DDRE=0xFF;   // PB5 out
    TCCR3A=0x82; // FAST PWM
    TCCR3B=0x1A; // 8분주=0.5usec
    
    DDRA = 0xFF;                // 포트 A 출력(서보모터)                 
    DDRB = 0xFF;                // 포트 B 출력 설정
    DDRD = 0b11111101;          // 포트 D 출력 설정(PD1만 입력)
    DDRG = 0xFF;                // 포트 G 출력 설정
    DDRE = 0xFF;            // 포트 E 출력 설정(부저)
    
    PORTB = 0x0;
    PORTD = 0x0;  
    PORTE = 0b00000100;

   // 블루투스 부분
   UCSR1A = 0x0;
    UCSR1B = 0b00010000;         // 수신 인에이블 RXEN0=1
    UCSR1C = 0b00000110;         // 비동기 데이터 8비트 모드  
    UBRR1H = 0;                  // X-TAL = 16MHz 일때, BAUD = 9600    
    UBRR1L = 103;

    TRIGGER = 0;

    TIMSK = 0b01000001;         // TOIE0=1, TOIE2=1, 타/카0/2 오버플로우 인터럽트 인에이블
    TCCR0 = 0b00000111;         // 타/카0 일반모드, 1024분주
    ASSR = 0x0;
    TCNT0 = 176;                // 1/16us * 1024분주 * (256 - 176) = 5.12ms
    
    TCCR2 = 0b00000000;         
    TCNT2 = 138;                // 1/16us * 8분주 * (256 - 138) = 58us

    SREG = 0x80;
    while(1){
        cnt = 0;   
        DDRC = 0xFF;
        TRIGGER = 1;                   // 초음파 센서  ECHO신호 출력  
        delay_us(15);
        TRIGGER = 0;
        while(PIND.1 == 0);             // ECHO = 1 일 때까지 대기
        TCNT2 = 138;
        TCCR2 = 0b00000010;         

        while(PIND.1 != 0) {            // ECHO = 0 일 떄까찌 대기
            if(cnt > 300) break;        // 3m보다 크면 측정 종료
        }  
        TCCR2 = 0b00000000;
        
      while((UCSR1A & 0x80) == 0x0){};
        rd = UDR1;                          // 수신
      connect = rd;

      if(connect == 'D')
      {
         for(i=0;i<25;i++){ PORTA.0=1; delay_us(2400); PORTA.0=0; delay_ms(20); } // 90도
         // 부저로 비상벨 작동
         for(i =0; i<3000; i++){PORTE = 0xFF; delay_us(10); PORTE = 0x00; delay_us(10);}

      }
      else if(connect == 'C') // 블루투스 비콘 연결시 작동부
      {
         // 서보모터 0으로 초기화하여 작동가능하게 변경
         for(i=0;i<25;i++){ PORTA.0=1; delay_us(1500); PORTA.0=0; delay_ms(20); }   // 0도
         // 거리가 30 보다 이하면 서보모터 브레이크
         if (cnt>2 && cnt<=30)
         {
            dist = cnt;
            for(i=0;i<25;i++){ PORTA.0=1; delay_us(2400); PORTA.0=0; delay_ms(20); } // 90도
            delay_ms(3000);
            // 다시 원상태로 뺄 수 있도록 다시 0도로 초기화
            for(i=0;i<25;i++){ PORTA.0=1; delay_us(1500); PORTA.0=0; delay_ms(20); }   // 0도
         }
         else if(cnt>2 && cnt<=60)
         {
            dist = cnt;
            // 부저로 위험 알림
            for(i =0; i<1000; i++){PORTE = 0xFF; delay_us(100); PORTE = 0x00; delay_us(100);}
         }
         else{
            dist = 300;
            for(i=0;i<25;i++){ PORTA.0=1; delay_us(1500); PORTA.0=0; delay_ms(20); }   // 0도
         }
      }
        Distance_out();
        delay_us(100);  
    }
} 

// 표시할 새 거리측정값
void Distance_out(void)
{                   
    int  buf;
    
    N1000 = dist / 1000;             // m 10자리 추출
    buf = dist % 1000;

    N100 = buf / 100;               // m 1자리 추출
    buf = buf % 100;          
    
    N10 = buf / 10;                 // cm 10자리 추출
    N1 = buf % 10;                  // cm 1자리 추출    
} 

// 거리측정값 표시(5.12ms마다 세그먼트 한 개씩 표시)
// 1/16us * 1024 * (256 -176) = 5.12ms
interrupt [TIM0_OVF] void time0(void)
{
    unsigned char pat;
    
    TCNT0 = 176;
                                     
    // 표시할 위치의 표시값
    if(dsp_no == 0) pat = N1;
    else if(dsp_no == 1) pat = N10;
    else if(dsp_no == 2) pat = N100;
    else pat = N1000;
                           
    // 7-세그먼트 표시
    PORTG = seg_on[dsp_no];                                 // 표시할 7-Segment ON
    PORTB = (seg_pat[pat] & 0x70) | (PORTB & 0x0F);         // e, f, g 부분 표시  -> PB4-PB6
    PORTD = ((seg_pat[pat] & 0x0F) << 4) | (PORTD & 0x0F);  // a, b, c, d 부분 표시  -> PD4-PD6  
    dsp_no = (dsp_no + 1) % 4; 
}

// 초음파 센서 거리 측정
// 1/16us * 8 * (256 -138) = 59us
interrupt [TIM2_OVF] void time2(void)
{
    TCNT2 = 138;
    cnt++;
}
