```
    #include <Wire.h>
    #include <Ethernet.h>
    #include <SPI.h>
    uint8_t sensor_addr = 0x42;

    //************ REGISTRADORES **********************//

    #define REG_COM1    0x04    /* Control 1 */

    #define REG_COM7    0x12    /* Control 7 */

    #define   COM7_RESET      0x80    /* Register reset */

    #define   COM7_FMT_MASK   0x38

    #define   COM7_FMT_VGA    0x00

    #define   COM7_FMT_CIF    0x20    /* CIF format */

    #define   COM7_FMT_QVGA   0x10    /* QVGA format */

    #define   COM7_FMT_QCIF   0x08    /* QCIF format */

    #define   COM7_RGB    0x04    /* bits 0 and 2 - RGB format */

    #define   COM7_YUV    0x00    /* YUV */

    #define   COM7_BAYER      0x01    /* Bayer format */

    #define   COM7_PBAYER     0x05    /* "Processed bayer" */

    #define REG_RGB444  0x8c    /* RGB 444 control */

    #define   R444_ENABLE     0x02    /* Turn on RGB444, overrides 5x5 */

    #define   R444_RGBX   0x01    /* Empty nibble at end */

    #define REG_COM9    0x14    /* Control 9  - gain ceiling */

    #define REG_COM10   0x15    /* Control 10 */

    #define REG_COM13   0x3d    /* Control 13 */

    #define   COM13_GAMMA     0x80    /* Gamma enable */

    #define   COM13_UVSAT     0x40    /* UV saturation auto adjustment */

    #define   COM13_UVSWAP    0x01    /* V before U - w/TSLB */

    #define REG_COM15   0x40    /* Control 15 */

    #define   COM15_R10F0     0x00    /* Data range 10 to F0 */

    #define   COM15_R01FE     0x80    /*            01 to FE */

    #define   COM15_R00FF     0xc0    /*            00 to FF */

    #define   COM15_RGB565    0x10    /* RGB565 output */

    #define   COM15_RGB555    0x30    /* RGB555 output */

    #define REG_COM11   0x3b    /* Control 11 */

    #define   COM11_NIGHT     0x80    /* NIght mode enable */

    #define   COM11_NMFR      0x60    /* Two bit NM frame rate */

    #define   COM11_HZAUTO    0x10    /* Auto detect 50/60 Hz */

    #define   COM11_50HZ      0x08    /* Manual 50Hz select */

    #define   COM11_EXP   0x02

    #define REG_COM16   0x41    /* Control 16 */

    #define   COM16_AWBGAIN   0x08    /* AWB gain enable */

    #define REG_COM17       0x42    /* Control 17 */

    #define COM17_AECWIN    0xc0    /* AEC window - must match COM4 */

    #define COM17_CBAR      0x08    /* DSP Color bar */

    #define REG_TSLB    0x3a    /* lots of stuff */

    #define   TSLB_YLAST      0x04    /* UYVY or VYUY - see com13 */

    #define MTX1            0x4f    /* Matrix Coefficient 1 */

    #define MTX2            0x50    /* Matrix Coefficient 2 */

    #define MTX3            0x51    /* Matrix Coefficient 3 */

    #define MTX4            0x52    /* Matrix Coefficient 4 */

    #define MTX5            0x53    /* Matrix Coefficient 5 */

    #define MTX6            0x54    /* Matrix Coefficient 6 */

    #define REG_CONTRAS     0x56    /* Contrast control */

    #define MTXS            0x58    /* Matrix Coefficient Sign */

    #define AWBC7           0x59    /* AWB Control 7 */

    #define AWBC8           0x5a    /* AWB Control 8 */

    #define AWBC9           0x5b    /* AWB Control 9 */

    #define AWBC10          0x5c    /* AWB Control 10 */

    #define AWBC11          0x5d    /* AWB Control 11 */

    #define AWBC12          0x5e    /* AWB Control 12 */

    #define REG_GFIX        0x69    /* Fix gain control */

    #define GGAIN           0x6a    /* G Channel AWB Gain */

    #define DBLV            0x6b    

    #define AWBCTR3         0x6c    /* AWB (Automatic White Balance) Control 3 */

    #define AWBCTR2         0x6d    /* AWB Control 2 */

    #define AWBCTR1         0x6e    /* AWB Control 1 */

    #define AWBCTR0         0x6f    /* AWB Control 0 */

    #define REG_COM8    0x13    /* Control 8 */

    #define   COM8_FASTAEC    0x80    /* Enable fast AGC/AEC (Automatic Gain Control - Automatic Exposure Control)*/

    #define   COM8_AECSTEP    0x40    /* Unlimited AEC step size */

    #define   COM8_BFILT      0x20    /* Band filter enable */

    #define   COM8_AGC    0x04    /* Auto gain enable */

    #define   COM8_AWB    0x02    /* White balance enable */

    #define   COM8_AEC    0x01    /* Auto exposure enable */

    #define REG_COM3        0x0c    /* Control 3 */

    #define COM3_SWAP       0x40    /* Byte swap */

    #define COM3_SCALEEN    0x08    /* Enable scaling */

    #define COM3_DCWEN      0x04    /* Enable downsamp/crop/window */

    #define REG_BRIGHT      0x55    /* Brightness */

    #define REG_COM14      0x3E    

    //*************************************************************************//

    byte mac[] = {
      0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED }; // MAC do arduino
    IPAddress ip(192,168,0,120); // IP do arduino
    byte server[] = {
      192,168,0,105}; //Endereço do servidor



    void setup (){

      DDRC&=~15;//low d0-d3 camera  PORTC (pino A0 ao pino A5) (11110000) A0 a A3 como entrada e A4 (SDA) e A5 (SCL) como saídas - essa linha tem a mesma função de pinMode que declara quem 
      //é pino de saída e de entrada. Se for 0 é entrada e se for 1 é saída. O valor é não 15 (acima).                 
      DDRD&=~255;//d7-d4 and interupt pins PORTD (pino 0 ao pino 7) (0000 d7 ao d4           0=d3=vsync 0=d2=pclock               0=d1=href   1=d0=button)

      DDRB&=~1;

      Serial.begin(57600);
      Wire.begin();

      UCSR0B = (1<<RXEN0)|(1<<TXEN0);//Enable receiver and transmitter (habilita bit 4 e bit 3)

      UCSR0C=6;// B (00000110) async (b7 e b6 = 00); (b3 = 0) 1 stop bit; (b2 e b1 = 1)  8bit char; no parity bits (b5 e b4 = 00); (b0 = 0)clock polaridade

      //set up twi for 100khz
      TWSR&=~3;//disable prescaler for TWI
      TWBR=72;//set to 100khz

      //SET UP CAMERA
      wrReg(0x15,32);//pclk does not toggle on HBLANK COM10 vsync falling
      //wrReg(0x15,0);//pclk does not toggle on HBLANK COM10 vsync falling
      wrReg(REG_RGB444, 0x00);             // Disable RGB444
      wrReg(REG_COM11,226);//enable nigh mode 1/8 frame rate COM11*/
      wrReg(REG_TSLB,0x04);                // 0D = UYVY  04 = YUYV     
      wrReg(REG_COM13,0x88);               // connect to REG_TSLB
      //definição para o RGB 565
      wrReg(REG_COM7, 0x04);           // RGB + color bar disable 
      wrReg(REG_COM15, 0xD0);          // Set rgb565 with Full range    0xD0

        wrReg(REG_COM3,0);    // REG_COM3

      wrReg(0x32,0xb6);        // HREF   
      wrReg(0x17,0x13);        // HSTART
      wrReg(0x18,0x01);        // HSTOP
      wrReg(0x19,0x02);        // VSTART
      wrReg(0x1a,0x7a);        // VSTOP
      wrReg(0x03,0x0a);        // VREF

      wrReg(REG_COM1, 0x00);



      //COLOR SETTING
      wrReg(REG_COM8,0x8F);        // AGC AWB AEC Unlimited step size
      wrReg(0xAA,0x14);            // Average-based AEC algorithm
      wrReg(REG_BRIGHT,0x00);      // 0x00(Brightness 0) - 0x18(Brightness +1) - 0x98(Brightness -1)
      wrReg(REG_CONTRAS,0x40);     // 0x40(Contrast 0) - 0x50(Contrast +1) - 0x38(Contrast -1)
      // wrReg(0xB1,0xB1);            // Automatic Black level Calibration
      wrReg(0xB1,4);//really enable auto black level calibration
      wrReg(MTX1,0x80);        
      wrReg(MTX2,0x80);      
      wrReg(MTX3,0x00);        
      wrReg(MTX4,0x22);        
      wrReg(MTX5,0x5e);        
      wrReg(MTX6,0x80);        
      wrReg(MTXS,0x9e);        
      wrReg(AWBC7,0x88);
      wrReg(AWBC8,0x88);
      wrReg(AWBC9,0x44);
      wrReg(AWBC10,0x67);
      wrReg(AWBC11,0x49);
      wrReg(AWBC12,0x0e);
      wrReg(REG_GFIX,0x00);
      //wrReg(GGAIN,0x40);
      wrReg(AWBCTR3,0x0a);
      wrReg(AWBCTR2,0x55);
      wrReg(AWBCTR1,0x11);
      wrReg(AWBCTR0,0x9f);

      wrReg(0xb0,0x84);//not sure what this does
      wrReg(REG_COM16,COM16_AWBGAIN);//disable auto denoise and edge enhancment
      //wrReg(REG_COM16,0);
      wrReg(0x4C,0);//disable denoise
      wrReg(0x76,0);//disable denoise
      wrReg(0x77,0);//disable denoise
      wrReg(0x7B,4);//brighten up shadows a bit end point 4
      wrReg(0x7C,8);//brighten up shadows a bit end point 8
      //wrReg(0x88,238);//darken highligts end point 176
      //wrReg(0x89,211);//try to get more highlight detail
      //wrReg(0x7A,60);//slope
      //wrReg(0x26,0xB4);//lower maxium stable operating range for AEC
      //hueSatMatrix(0,100);
      //ov7670_store_cmatrix();
      //wrReg(0x20,12);//set ADC range to 1.5x
      wrReg(REG_COM9,0x6A);//max gain to 128x
      wrReg(0x11,9);//using scaler for divider
      delay(100);//i2c uses interupts to write data wait for all bytes to be written



      //            #ifdef qqvga || ifdef qvga

      wrReg(REG_COM3,4);    // REG_COM3 

      //    #ifdef qqvga

      wrReg(REG_COM14, 0x1a);          // divide by 4

      wrReg(0x72, 0x22);               // downsample by 4

      wrReg(0x73, 0xf2);               // divide by 4

      wrReg(0x17,0x16);

      wrReg(0x1A,0x04);

      wrReg(0x32,0xa4);           

      wrReg(0x19,0x02);

      wrReg(0x18,0x7a);

      wrReg(0x03,0x0a);




      wrReg(REG_COM1, 0x00);

      // COLOR SETTING



      wrReg(REG_COM8,0x8F);        // AGC AWB AEC Unlimited step size

      wrReg(0xAA,0x14);            // Average-based AEC algorithm

      wrReg(REG_BRIGHT,0x00);      // 0x00(Brightness 0) - 0x18(Brightness +1) - 0x98(Brightness -1)

      wrReg(REG_CONTRAS,0x40);     // 0x40(Contrast 0) - 0x50(Contrast +1) - 0x38(Contrast -1)

      // wrReg(0xB1,0xB1);            // Automatic Black level Calibration

      wrReg(0xB1,4);//really enable auto black level calibration

      wrReg(MTX1,0x80);        

      wrReg(MTX2,0x80);      

      wrReg(MTX3,0x00);        

      wrReg(MTX4,0x22);        

      wrReg(MTX5,0x5e);        

      wrReg(MTX6,0x80);        

      wrReg(MTXS,0x9e);        

      wrReg(AWBC7,0x88);

      wrReg(AWBC8,0x88);

      wrReg(AWBC9,0x44);

      wrReg(AWBC10,0x67);

      wrReg(AWBC11,0x49);

      wrReg(AWBC12,0x0e);

      wrReg(REG_GFIX,0x00);

      //wrReg(GGAIN,0x40);

      wrReg(AWBCTR3,0x0a);

      wrReg(AWBCTR2,0x55);

      wrReg(AWBCTR1,0x11);

      wrReg(AWBCTR0,0x9f);



      wrReg(0xb0,0x84);//not sure what this does

      wrReg(REG_COM16,COM16_AWBGAIN);//disable auto denoise and edge enhancment

      //wrReg(REG_COM16,0);

      wrReg(0x4C,0);//disable denoise

      wrReg(0x76,0);//disable denoise

      wrReg(0x77,0);//disable denoise

      wrReg(0x7B,4);//brighten up shadows a bit end point 4

        wrReg(0x7C,8);//brighten up shadows a bit end point 8

        //wrReg(0x88,238);//darken highligts end point 176

      //wrReg(0x89,211);//try to get more highlight detail

      //wrReg(0x7A,60);//slope

      //wrReg(0x26,0xB4);//lower maxium stable operating range for AEC

      //hueSatMatrix(0,100);

      //ov7670_store_cmatrix();

      //wrReg(0x20,12);//set ADC range to 1.5x

      wrReg(REG_COM9,0x6A);//max gain to 128x

      //    #ifdef qvga

      wrReg(0x11,4);//using scaler for divider

      pinMode(4,OUTPUT);
      digitalWrite(4,HIGH);
      Serial.println("Iniciando conexao");
      Ethernet.begin(mac, ip);
      Serial.println("Por favor, espere.");
      delay(1000);

    }


    EthernetClient client;
    void loop (){

    connectWeb(7000,server);
    foto(480,640);

    }


    byte wrReg(int regID, int regDat )

    { 

      Wire.beginTransmission(sensor_addr >> 1);
      Wire.write(regID & 0x00FF);   
      Wire.write(regDat & 0x00FF);  
      if(Wire.endTransmission())
      {
        return 0; 
        PORTB|=32;
        while (1) {
        }
      }
      delay(1);
      return(1);
    }

    uint16_t d;


    void foto(uint16_t linha, uint16_t coluna)
    { 
      while (!(PIND&8)) {
      }//wait for high
      while ((PIND&8)) {
      }//wait for low
      //uint16_t a =millis(); //teste de tempo
      if (linha != 0)
      {
        //while (!(PINB&1)) {}//wait for high
        //while ((PINB&1)) {}//wait for low
        //while (!(PINB&1)) {}//wait for high

        while (linha--)
        {

          while (!(PIND&4)) {
          }//wait for high
          while ((PIND&4)) {
          }//wait for low
          //      int a = millis(); //teste de tempo

          uint16_t byte1, byte2;
          uint8_t buf[512];

          uint16_t  coluna_temp=coluna;
          while (coluna--){

            for(int i=0; i<2;i++){
              while (!(PIND&4)) {
              }//wait for high
              if(i==0){             
                byte1= (uint8_t)(PINC&15)|(PIND&240); //B 00001111 (A0 a A3) ou 11110000 (d7 ao d4)
              }
              else{
                byte2= (uint8_t)(PINC&15)|(PIND&240);

                //((byte1<<8)|byte2);
               /* for(int x=0;x<512;x++){
                  buf[x]= (((byte1<<8)|byte2),BIN);
                  d+=buf[x];        
                }*/
            sendMsg(client, ((byte1<<8)|byte2));    
                //Serial.println(d);

                // Serial.println(((byte1<<8)|byte2),BIN); //B 00001111 (A0 a A3) ou 11110000 (d7 ao d4)
                //teste++;
              }
              while ((PIND&4)) {
              }//wait for low
            }

          }
          //                                        int b = millis(); //teste de tempo
          //  Serial.println((b-a)); //teste de tempo
          coluna=coluna_temp;
        }

        //while ((PIND&1)) {}//wait for low



      }
      //uint16_t b =millis(); //teste de tempo
      //Serial.println((b-a)); //teste de tempo
     // return teste;

    }



    void connectWeb(int port, byte ipServer[]){

      if (client.connect(ipServer,port)){
        Serial.println("Conectado");
        //essa e a parte que da erro. Perceba que estou enviando a mensagem
        //aqui, mas a intençao e enviar na funcao de tirar a foto.Aqui e teste
       // sendMsg(client, 800);

      }else{
        Serial.println("Conexao falhou");
      }

       if (!client.connected()) {
        Serial.println();
        Serial.println("disconnecting.");
        client.stop();

        // do nothing forevermore:
        while(true);
      }

    }

    //void sendMsg(EthernetClient client, uint8_t msg){
    //      client.println("valor "+String(msg));
    //}

    void sendMsg(EthernetClient client, uint32_t msg){
          client.println(msg,DEC);
    }
    ```
