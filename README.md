;==========================================================================
Time0                  Equ      -1000+4
Dark                   Equ      10H
;--------------------------------------------------------------------------
TimeRst                Bit      P1.5
TimeClk                Bit      P1.6
TimeDat                Bit      P1.7
;--------------------------------------------------------------------------
DisModePort0           Equ      6000H                  ;0:Light up
DisModePort1           Equ      0000H
DisCSPort0             Equ      2000H                  ;1:Light up
DisCSPort1             Equ      4000H
NoBlinkDat             Equ      0FFH
BlinkStartDat          Equ      0FEH
;===========================1302===========================================
Second                 Equ      40H
Minute                 Equ      41H
Hour                   Equ      42H
Day                    Equ      43H
Month                  Equ      44H
Date                   Equ      45H
Year                   Equ      46H
ProtReg                Equ      47H
;--------------------------------------------------------------------------
KeyInport              Equ      P1
KeyOutport             Equ      DisCsPort0
;==========================================================================
KeyValue               Data     21H
SysBits0               Data     22H
SysBits1               Data     23H
KeyPress               Bit      SysBits0.0
Keyrelease             Bit      SysBits0.1
;--------------------------------------------------------------------------
Time1MsSign            Bit      SysBits0.2
Time10MsSign           Bit      SysBits0.3
Time01sSign            Bit      SysBits0.4
Time1sSign             Bit      SysBits0.5
Time1mSign             Bit      SysBits0.6
Time1hSign             Bit      SysBits0.7
ModifyTime             Bit      SysBits1.0
NeedReadTime           Bit      SysBits1.1
Blink                  Equ      ModifyTime
;--------------------------------------------------------------------------
DisBuf0                Data     50H                  ;..29H
DisBuf1                Data     60H                  ;..31H
;--------------------------------------------------------------------------
Time1Ms                Data     32H
Time10Ms               Data     33H
Time01s                Data     34H
Time1s                 Data     35H
Time1m                 Data     36H
Time1h                 Data     37H
DisTime1s              Data     38H
DisTime1m              Data     39H
DisTime1h              Data     3AH
DisDate                Data	3BH
DisDay		       Data     3CH
DisMonth               Data     3DH
DisYear                Data     3EH
;--------------------------------------------------------------------------
ModifyBit0             Data     3Fh
ModifyBit1             Data     40H
;--------------------------------------------------------------------------
DisBit0                Data     41H
DisBit1                Data     42H
DisCS0                 Data     43H
DisCS1                 Data     44H
;--------------------------------------------------------------------------
WaitTime               Data     45H
;==========================================================================
                       Org      0000H
		       Ajmp     Main
;--------------------------------------------------------------------------
                       Org      000BH
		       Mov      TL0,#Low(Time0)
		       MOv      TH0,#High(Time0)
		       Ajmp     T0Entry
;==========================================================================
T0Entry:               SetB     Time1MsSign
		       Djnz     Time1Ms,Quit
		       Mov      Time1Ms,#10
		       
		       SetB     Time10MsSign
		       Djnz     Time10Ms,Quit
		       Mov      Time10Ms,#10

                       SetB     Time01sSign
		       Djnz     Time01s,Quit
		       Mov      Time01s,#10
		       
		       SetB     Time1sSign
		       Djnz     Time1s,Quit
		       Mov      Time1s,#10
		       
		       SetB     Time1mSign
		       Djnz     Time1m,Quit
		       Mov      Time1m,#10
		       
		       SetB     Time1hSign
		       Djnz     Time1h,Quit
		       Mov      Time1h,#10
		       
Quit:                  RetI
;==========================================================================
Init1302:              Clr      TimeRst
                       Clr      TimeClk
                       SetB     TimeRst
		       Ret
;-------------------------------------------------------------------------
InitDis:               Mov      DisBuf0,#0
                       Mov      DisBuf0+1,#0
		       Mov      DisBuf0+2,#0
		       Mov      DisBuf0+3,#0
		       Mov      DisBuf0+4,#0
		       Mov      DisBuf0+5,#0
		       Mov      DisBuf1,#0
		       Mov      DisBuf1+1,#Dark
		       Mov      DisBuf1+2,#0
		       Mov      DisBuf1+3,#0
		       Mov      DisBuf1+4,#0
		       Mov      DisBuf1+5,#0
		       Mov      DisBuf1+6,#0
		       Mov      DisBuf1+7,#0
                       Mov      DisBit0,#Disbuf0
		       Mov      DisBit1,#Disbuf1
		       Mov      DisCS0,#00000001B
                       Mov      DisCS1,#00000001B
		       Ret
;--------------------------------------------------------------------------
InitTime:              Mov      TMod,#00000001B        ;Use T0,Timer,No Gate,16 Bit
                       Mov      TL0,#Low(Time0)        ;Pre-Load Value
                       Mov      TH0,#High(Time0)
                       Setb     EA                     ;Enable T0 Int
                       Setb     ET0
                       Setb     TR0
		       
                       Clr      Time1MsSign
                       Clr      Time10MsSign
                       Clr      Time01sSign
		       Clr      Time1sSign
		       Clr      Time1mSign
		       Clr      Time1hSign
                       Mov      Time1Ms,#10
                       Mov      Time10Ms,#10
                       Mov      Time01s,#10
 		       Mov      Time1s,#10
 		       Mov      Time1m,#10
 		       Mov      Time1h,#10
		       
		       SetB     NeedReadTime
                       Ret
;--------------------------------------------------------------------------
InitKey:               Clr      KeyPress
                       SetB     KeyRelease
                       Clr      ModifyTime
		       Mov      WaitTime,#0
		       Ret
;--------------------------------------------------------------------------
KeyScan:               Clr      A
                       Mov      Dptr,#KeyOutPort
		       Movx     @Dptr,A
		       Mov      A,KeyInPort
		       Cpl      A
		       Anl      A,#0FH
		       Ret
;--------------------------------------------------------------------------
ReadKey:               Mov      R2,#0
KeyConfirmLoop:        Acall    KeyScan
		       Jnz      KeyNoRelease
                       SetB     KeyRelease
		       Ajmp     Quit2
KeyNoRelease:          Inc      R2
		       Cjne     R2,#3,KeyConfirmLoop
		       Jnb      KeyRelease,Quit2
		       
		       SetB     KeyPress
		       Clr      KeyRelease
		       Mov      WaitTime,#0
		       Mov      R3,#6
		       Mov      R2,#11011111B
ReadKeyLoop:	       Mov      A,R2
		       Mov      Dptr,#KeyOutport
		       Movx     @Dptr,A
		       
		       Mov      A,#0
		       Jnb      KeyInPort.0,ReadKeyOK
		       Mov      A,#6
		       Jnb      KeyInPort.1,ReadKeyOK
		       Mov      A,#12
		       Jnb      KeyInPort.2,ReadKeyOK
		       Mov      A,#18
		       Jnb      KeyInPort.3,ReadKeyOK
		       
		       Mov      A,R2
		       RR       A
		       Mov      R2,A
		       Djnz     R3,ReadKeyLoop
		       Ret
		       
ReadKeyOk:	       Add      A,R3
                       Dec      A
		       Mov      Dptr,#KeyTab
		       Movc     A,@A+Dptr
		       Mov      KeyValue,A
		       
Quit2:                 Ret
;--------------------------------------------------------------------------
KeyTab:                DB       10,00,11,12,13,14
                       DB       01,02,03,15,16,17
		       DB       04,05,06,18,19,20
		       DB       07,08,09,21,22,23
;--------------------------------------------------------------------------
DoKey:                 Jbc      KeyPress,$+4
                       Ret 
                       Mov      A,KeyValue
                       Cjne     A,#10,$+3
		       Jnc      DoFunc
		       
		       Jnb      ModifyTime,Quit4
                       Ajmp     KeyIsNum		       
		       
DoFunc:                Add      A,#-10
                       Rl       A
		       Mov      Dptr,#FuncTab
		       Jmp      @A+Dptr

FuncTab:               Ajmp     Func0
                       Ajmp     Func1
		       Ajmp     Func2
		       Ajmp     Func3
                       ;Ajmp     Func4
;		       Ajmp     Func5
;              	       Ajmp     Func6
;                       Ajmp     Func7
;		       Ajmp     Func8
;		       Ajmp     Func9
 ;                      Ajmp     Func10
;		       Ajmp     Func11
;		       Ajmp     Func12
 ;                      Ajmp     Func13
		       
Func0:                 Jb       ModifyTime,Quit4
                       SetB     ModifyTime
                       Mov      ModifyBit0,#BlinkStartDat
		       Mov      ModifyBit1,#NoBlinkDat
		       Clr      NeedReadTime
Quit4:                 Ret

Func1:                 Jnb      ModifyTime,Nokey
                       Mov      A,ModifyBit0
                       Cjne     A,#0FFH,ModifyHMS
		       Mov      A,ModifyBit1
		       SetB     C
		       Rlc      A
		       Jb       Acc.7,OutPutMB1
		       Mov      A,#NoBlinkDat
		       Mov      ModifyBit0,#BlinkStartDat
OutputMB1:             Mov      ModifyBit1,A
		       Ret
			
ModifyHMS:             Mov      A,ModifyBit0
                       SetB     C
                       Rlc      A
                       Jb       Acc.6,OutputMB0                                 ;还没到6位，直接输出
		       Mov      A,#NoBlinkDat
		       Mov      ModifyBit1,#BlinkStartDat
OutputMB0:             Mov      ModifyBit0,A
                       Ret
		       
Func2:                 Clr      ModifyTime
                       Acall    Unprot1302
		       Mov      B,DisTime1s
		       Mov      A,#Second
		       Acall    BATo1302
		       
		       Mov      B,DisTime1m
		       Mov      A,#Minute
		       Acall    BATo1302

		       Mov      B,DisTime1h
		       Mov      A,#Hour
		       Acall    BATo1302
		       
		       Mov      B,DisDay
		       Mov      A,#Day
		       Acall    BATo1302
		       
		       Mov      B,DisYear
		       Mov      A,#Year
		       Acall    BATo1302
		       
		       Mov      B,DisMonth
		       Mov      A,#Month
		       Acall    BATo1302
		       
                       Mov      B,DisDate
		       Mov      A,#Date
		       Acall    BATo1302

		       SetB     NeedReadTime
		       Acall    Prot1302
                       Ret
		       
Func3:                 Clr      ModifyTime
                       SetB     NeedReadTime   
		       
NoKey:                 Ret
;-------------------------------------------------------------------------
KeyIsNum:	       Mov      A,ModifyBit0
                       Jnb      Acc.0,DisNo0
                       Jnb      Acc.1,DisNo1
                       Jnb      Acc.2,DisNo2
		       Jnb      Acc.3,DisNo3
		       Jnb      Acc.4,DisNo4
		       Jnb      Acc.5,DisNo5
		       Ajmp     No678
		       
DisNo0:                Mov      A,KeyValue
		       Cjne     A,#1,$+3
		       Jnc      Quit7
                       Anl      DisTime1s,#0F0H
                       Orl      A,DisTime1s
		       Mov      DisTime1s,A
Quit7:                 Ret		       
		       
DisNo1:                Mov      A,KeyValue
		       Cjne     A,#6,$+3
		       Jnc      Quit7
                       Anl      DisTime1s,#8FH
		       Swap     A
                       Orl      A,DisTime1s
		       Mov      DisTime1s,A
                       Ret
		       
DisNo2:                Mov      A,KeyValue
                       Anl      DisTime1m,#0F0H
                       Orl      A,DisTime1m
		       Mov      DisTime1m,A
                       Ret		       
		       
DisNo3:                Mov      A,KeyValue
		       Cjne     A,#6,$+3
		       Jnc      Quit7
                       Anl      DisTime1m,#8FH
		       Swap     A
                       Orl      A,DisTime1m
		       Mov      DisTime1m,A
                       Ret		  
		       
DisNo4:                Mov      A,KeyValue
                       Anl      DisTime1h,#0F0H
                       Orl      A,DisTime1h
		       Mov      DisTime1h,A	     
		       Ret
		       
DisNo5:                Mov      A,KeyValue
		       Cjne     A,#3,$+3
		       Jnc      Quit7
                       Anl      DisTime1h,#8FH
		       Swap     A
                       Orl      A,DisTime1h
		       Cjne     A,#24H,$+3
		       Jnc      Quit7
		       Mov      DisTime1h,A
		       Ret

No678:   	       Mov      A,ModifyBit1
		       Jnb      Acc.0,DisNo6
		       Jnb      Acc.1,DisNo7
		       Jnb      Acc.2,DisNo8
		       Jnb      Acc.3,DisNo9
		       Jnb      Acc.4,DisNo10
		       Jnb      Acc.5,DisNo11
		       Jnb      Acc.6,DisNo12
		       Ajmp     Quit7
		       
DisNo6:                Mov      A,KeyValue
		       Cjne     A,#8,$+3
		       Jnc      Quit5
		       Mov      DisDate,A
		       Ret
		       
DisNo7:                Mov      A,KeyValue
                       Anl      DisDay,#0F0H
		       Orl      A,DisDay
		       Cjne     A,#32H,$+3
		       Jnc      Quit5
		       Mov      DisDay,A
		       
DisNo8:                Mov      A,KeyValue
                       Cjne     A,#4,$+3
		       Jnc      Quit5
		       Anl      DisDay,#8FH
		       Swap     A
		       Orl      A,DisDay
		       Cjne     A,#32H,$+3
		       Jnc      Quit5
		       Mov      DisDay,A
		       Ret		       
                     
DisNo9:                Mov      A,KeyValue
		       Jnc      Quit5
                       Anl      DisMonth,#0F0H
		       Orl      A,DisMonth
		       Cjne     A,#13H,$+3
		       Jnc      Quit5
		       Mov      DisMonth,A
		       Ret
		    
DisNo10:               Mov      A,Keyvalue
                       Cjne     A,#2,$+3
		       Jnc      Quit5
		       Anl      DisMonth,#8FH
		       Swap     A
		       Orl      A,DisMonth
		       Cjne     A,#13H,$+3
		       Jnc      Quit5
		       Mov      A,DisMonth
		       Ret
		       
DisNo11:               Mov      A,KeyValue
                       Anl      DisYear,#0F0H
		       Orl      A,DisYear
		       Mov      DisYear,A
		       Ret
		       
DisNo12:               Mov      A,KeyValue
                       Anl      DisYear,#0FH
		       Swap     A
		       Orl      A,DisYear
		       Mov      DisYear,A

Quit5:    	       Ret
;----------------------------------------------------------------------------
IncWaitTime:           Jnb      ModifyTime,Quit9
                       Inc      WaitTime
		       Mov      A,Waittime
		       Cjne     A,#4,Quit9
		       Acall    Func3
Quit9:                 Ret
;============================================================================
ReadTime:              Jnb      NeedReadTime,Quit6
                       
		       Mov      A,#Second
		       ACall    GotDat
		       Mov      DisTime1s,A
		       
		       Mov      A,#Minute
		       Acall    GotDat
		       Mov      DisTime1m,A
		       
		       Mov      A,#Hour
		       Acall    GotDat
		       Mov      DisTime1h,A
                       
                       Mov      A,#Date
		       Acall    GotDat
		       Mov      DisDate,A

                       Mov      A,#Day
		       Acall    GotDat
		       Mov      DisDay,A
                
                       Mov      A,#Month
		       Acall    GotDat
		       Mov      DisMonth,A
		     
		       Mov      A,#Year
		       Acall    GotDat
		       Mov      DisYear,A
		       
Quit6:  	       Ret
;--------------------------------------------------------------------------
ShowTime:              Mov      A,DisTime1s
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf0,B
		       Mov      Disbuf0+1,A
		       
		       Mov      A,DisTime1m
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf0+2,B
		       Mov      Disbuf0+3,A
		       
		       Mov      A,DisTime1h
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf0+4,B
		       Mov      Disbuf0+5,A
		       
		       Mov      A,DisDate
		       Mov      Disbuf1,A
		       
		       Mov      A,DisDay
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf1+1,B
		       Mov      Disbuf1+2,A
		       
		       Mov      A,DisMonth
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf1+3,B
		       Mov      Disbuf1+4,A
		       
		       Mov      A,DisYear
                       Mov      B,#16
		       Div      AB
		       Mov      Disbuf1+5,B
		       Mov      Disbuf1+6,A
		       
		       Ret
;===============================================================================
GotDat:                SetB     TimeRst
                       SetB     C
		       Rlc      A
		       ACall    ATo1302
                       Mov      R2,#8
GotDatLoop:            Mov      C,TimeDat
                       Rrc      A
		       SetB     TimeClk		       
		       Clr      TimeClk
		       Mov      C,TimeDat
		       Djnz     R2,GotDatLoop
		       Clr      TimeRst
		       Ret
;-------------------------------------------------------------------------------
Prot1302:              Mov      B,#0FFH
                       Ajmp     GiveA
;-------------------------------------------------------------------------------
Unprot1302:            Mov      B,#0
GiveA:   	       Mov      A,#ProtReg
;-------------------------------------------------------------------------------
BATo1302:              SetB     TimeRst
		       Rl       A
                       Acall    ATo1302
		       Xch      A,B
		       Acall    ATo1302
		       Clr      TimeRst
		       Ret
;-------------------------------------------------------------------------------               
ATo1302:               Mov      R2,#8
ATo1302Loop:           Rrc      A
		       Mov      TimeDat,C
		       SetB     TimeClk
		       Clr      TimeClk
		       Djnz     R2,ATo1302Loop
		       Ret
;================================================================================
DisplayTime:           Mov      A,DisCS0
                       Jnb      Blink,NoBlink
                       Mov      A,Time01s
                       Cjne     A,#5,$+3 
		       Mov      A,DisCS0
		       Jnc      NoBlink
		       Anl      A,ModifyBit0
NoBlink:	       Mov      Dptr,#DisCsPort0
		       Movx     @Dptr,A
		       Mov      A,DisCS0
		       Rl       A
		       Mov      DisCS0,A
		       
		       Mov      R0,DisBit0
		       Mov      A,@R0
		       Mov      Dptr,#DisTab
		       Movc     A,@A+Dptr
		       Mov      Dptr,#DisModePort0
		       Movx     @Dptr,A
		       Inc      R0
		       Mov      DisBit0,R0
		       Cjne     R0,#Disbuf0+6,Quit1
		       Mov      DisCS0,#00000001B
		       Mov      DisBit0,#DisBuf0
Quit1:                 Ret      
;-------------------------------------------------------------------------------
DisplayDate:           Mov      A,DisCS1
                       Jnb      Blink,NoBlink1
                       Mov      A,Time01s
                       Cjne     A,#5,$+3 
		       Mov      A,DisCS1
		       Jnc      NoBlink1
		       Anl      A,ModifyBit1
NoBlink1:	       Mov      Dptr,#DisCsPort1
		       Movx     @Dptr,A
		       Mov      A,DisCS1
		       Rl       A
		       Mov      DisCS1,A
		       
		       Mov      R0,DisBit1
		       Mov      A,@R0
		       Mov      Dptr,#DisTab
		       Movc     A,@A+Dptr
		       Mov      Dptr,#DisModePort1
		       Movx     @Dptr,A
		       Inc      R0
		       Mov      DisBit1,R0
		       Cjne     R0,#Disbuf1+8,Quit8
		       Mov      DisCS1,#00000001B
		       Mov      DisBit1,#DisBuf1
Quit8:                 Ret
;---------------------------------------------------------------------------------
;                      Code   00h,  01h,  02h,  03h, 04h,  05h,  06h, 07h
;                      Char   "0",  "1",  "2",  "3", "4",  "5",  "6", "7"
DisTab:                DB    0c0h, 0f9h, 0a4h, 0b0h, 99h,  92h,  82h, 0f8h

;                      Code   08h, 09h,  0ah,  0bh,  0ch , 0dh , 0eh, 0fh
;                      Char   "8", "9",  "a",  "b",  "c" , "d" , "e", "f"
                       DB     80h, 90h,  88h,  83h,  0c6h, 0a1h, 86h, 8eh

;                      Code   10h   11h
;                      Char   " "   "H"
                       DB     0FFH, 89H,  00H
;==========================================================================
Main:                  Mov      Sp,#07
                       Acall    InitTime
		       Acall    InitDis
		       Acall    Init1302
		       Acall    InitKey
		       
MainLoop0:             Jbc      Time1MsSign,ProgramFor1Ms
MainLoop1:             Jbc      Time10MsSign,ProgramFor10Ms
MainLoop2:             Jbc      Time01sSign,ProgramFor01s
MainLoop3:             Jbc      Time1sSign,ProgramFor1s
                       Ajmp     MainLoop0
		       
ProgramFor1Ms:         Acall    DisplayTime
                       Acall    DisplayDate
		       Ajmp     MainLoop1
		       
ProgramFor10Ms:        Acall    ReadTime
                       Acall    ShowTime
		       Ajmp     MainLoop2
		       
ProgramFor01s:         Acall    ReadKey
		       Acall    DoKey
                       
		       Ajmp     MainLoop3

ProgramFor1s:          Acall    IncWaitTime
                       Ajmp     MainLoop0
;==========================================================================
                       End
