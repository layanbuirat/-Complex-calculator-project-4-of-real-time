PROCESSOR 16F877A
	#include <p16f877a.inc>
	__CONFIG _CP_OFF & _WDT_OFF & _BODEN_ON & _PWRTE_ON & _XT_OSC & _LVP_OFF

;==============================================================================
; 1. REGISTER AND CONSTANT DEFINITIONS
;==============================================================================

    cblock  0x20        ; Start of General Purpose Register block
    ; --- System State & Control ---
    CALC_STATE          ; Main state machine variable
    DISP_SCREEN         ; Display mode: 0=Result, 1=Num1, 2=Num2

    ; --- Number Storage Buffers ---
    NUM1_INT: 6        ; Buffer for integer part of first number
    NUM1_FRAC: 6       ; Buffer for fractional part of first number
    NUM2_INT: 6        ; Buffer for integer part of second number
    NUM2_FRAC: 6       ; Buffer for fractional part of second number
    CALC_RESULT: 24    ; Buffer for co-processor result

    ; --- User Input Handling ---
    CURSOR_POS         ; Current digit being edited
    LAST_CURSOR        ; Previous cursor position
    DOUBLE_TAP_FLAG    ; Flag for double-click detection

    ; --- ISR and Temporary Storage ---
    ISR_W_TEMP         ; W register save for ISR
    ISR_STAT_TEMP      ; STATUS register save for ISR
    TEMP_COUNT         ; Temporary counter
    WORK_REG_A         ; Temporary register A
    WORK_REG_B         ; Temporary register B
    INPUT_TIMER        ; Timer for input loops
    endc

; --- State Constants ---
#define STATE_INIT         0
#define STATE_NUM1_INPUT   1
#define STATE_NUM2_INPUT   2
#define STATE_SEND_DATA    3
#define STATE_RECV_DATA    4
#define STATE_DISP_RESULT  5

;==============================================================================
; PROGRAM VECTORS
;==============================================================================

	ORG 0x00            ; Reset Vector
	goto Main_Entry

	ORG 0x04            ; Interrupt Vector
	goto ISR_Service

;==============================================================================
; UTILITY AND HELPER ROUTINES
;==============================================================================

; --- Delay Routines ---
Pause_1ms:
    MOVLW   d'249'
    MOVWF   TEMP_COUNT
Pause1ms_Loop:
    NOP
    DECFSZ  TEMP_COUNT, F
    GOTO    Pause1ms_Loop
    RETURN

Pause_500ms:
    BCF     STATUS, RP0
    BCF     STATUS, RP1
    MOVLW   0xFF
    MOVWF   0x7D
    MOVLW   0x83
    MOVWF   0x7E
    MOVLW   0x02
    MOVWF   0x7F
Pause500ms_L1:
    DECFSZ  0x7D, F
    GOTO    Pause500ms_L1
Pause500ms_L2:
    DECFSZ  0x7E, F
    GOTO    Pause500ms_L1
Pause500ms_L3:
    DECFSZ  0x7F, F
    GOTO    Pause500ms_L2
    RETURN

Pause_2s:
    CALL    Pause_500ms
    CALL    Pause_500ms
    CALL    Pause_500ms
    CALL    Pause_500ms
    RETURN

; --- Buffer Management ---
Reset_NUM2_Buffer:
    MOVLW   NUM2_INT
    MOVWF   FSR
    MOVLW   0x0C
    MOVWF   TEMP_COUNT
Reset_NUM2_Loop:
    CLRF    INDF
    INCF    FSR, F
    DECFSZ  TEMP_COUNT, F
    GOTO    Reset_NUM2_Loop
    RETURN

Reset_Result_Buffer:
    MOVLW   CALC_RESULT
    MOVWF   FSR
    MOVLW   0x18
    MOVWF   TEMP_COUNT
Reset_Result_Loop:
    CLRF    INDF
    INCF    FSR, F
    DECFSZ  TEMP_COUNT, F
    GOTO    Reset_Result_Loop
    RETURN

; --- Input Detection ---
Check_Double_Tap:
    CLRF    DOUBLE_TAP_FLAG
    MOVLW   0xFF
    MOVWF   WORK_REG_B
    BTFSS   PORTB, 0
    GOTO    $-1
Check_Tap_Loop:
    BTFSS   PORTB, 0
    GOTO    Set_Double_Tap
    CALL    Pause_1ms
    CALL    Pause_1ms
    DECFSZ  WORK_REG_B, F
    GOTO    Check_Tap_Loop
    RETURN
Set_Double_Tap:
    MOVLW   0x01
    MOVWF   DOUBLE_TAP_FLAG
    RETURN

; --- Communication Routines ---
Transmit_To_CoProc:
    MOVLW   NUM1_INT  
    MOVWF   FSR
    MOVLW   0x18
    MOVWF   WORK_REG_A
Send_Loop:
    MOVF    INDF, W
    ANDLW   0x0F
    MOVWF   PORTC
    BSF     PORTD, 0
    NOP
    BCF     PORTD, 0
Wait_CoProc_ACK:
    BTFSS   PORTB, 5
    GOTO    Wait_CoProc_ACK
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    INCF    FSR, F
    DECFSZ  WORK_REG_A, F
    GOTO    Send_Loop
    RETURN

Receive_From_CoProc:
    BANKSEL TRISC
    MOVLW   0xFF
    MOVWF   TRISC
    BANKSEL FSR
    MOVLW   CALC_RESULT
    MOVWF   FSR
    GOTO    Recv_Loop
Recv_Loop:
    BTFSS   PORTB, 2
    GOTO    $-1
    MOVF    PORTC, W
    ANDLW   0x0F
    MOVWF   INDF
    BSF     PORTB, 1
    NOP
    NOP
    BCF     PORTB, 1
    INCF    FSR, F
    MOVF    FSR, W
    SUBLW   0x46
    BTFSS   STATUS, Z
    GOTO    Recv_Loop
    GOTO    Switch_To_Result_Mode

;==============================================================================
; LCD DISPLAY ROUTINES (RENAMED ONLY, INTERNALS UNCHANGED)
;==============================================================================

    INCLUDE "LCDIS.INC"

Clear_LCD:
    MOVLW   0x01
    BCF     Select, RS
    CALL    send
    MOVLW   0x80
    BCF     Select, RS
    CALL    send
    BSF     Select, RS
    RETURN

Show_Welcome:
    CALL    Clear_LCD
    MOVLW   'W'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   'l'
    CALL    send
    MOVLW   'c'
    CALL    send
    MOVLW   'o'
    CALL    send
    MOVLW   'm'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   ' '
    CALL    send
    MOVLW   't'
    CALL    send
    MOVLW   'o'
    CALL    send
    BCF     Select, RS
    MOVLW   0xC0
    CALL    send
    BSF     Select, RS
    MOVLW   'D'
    CALL    send
    MOVLW   'i'
    CALL    send
    MOVLW   'v'
    CALL    send
    MOVLW   'i'
    CALL    send
    MOVLW   's'
    CALL    send
    MOVLW   'i'
    CALL    send
    MOVLW   'o'
    CALL    send
    MOVLW   'n'
    CALL    send
    RETURN

Display_NUM1:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVLW   'N'
    CALL    send
    MOVLW   'u'
    CALL    send
    MOVLW   'm'
    CALL    send
    MOVLW   'b'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   'r'
    CALL    send
    MOVLW   ' '
    CALL    send
    MOVLW   '1'
    CALL    send
    BCF     Select, RS
    MOVLW   0xC0
    CALL    send
    BSF     Select, RS
    MOVF    NUM1_INT+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+5, W
    ADDLW   '0'
    CALL    send
    MOVLW   '.'
    CALL    send
    MOVF    NUM1_FRAC+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+5, W
    ADDLW   '0'
    CALL    send
    BCF     Select, RS
    MOVLW   0x0C
    CALL    send
    MOVLW   0x0E
    CALL    send
    MOVF    CURSOR_POS, W
    SUBLW   0x0D
    MOVWF   WORK_REG_A
    MOVF    CURSOR_POS, W
    SUBLW   5
    BTFSS   STATUS, C
    DECF    WORK_REG_A, F
Move_Cursor_NUM1:
    BCF     Select, RS
    MOVLW   0x10
    CALL    send
    DECFSZ  WORK_REG_A, F
    GOTO    Move_Cursor_NUM1
    RETURN

Display_NUM2_Header:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVLW   'N'
    CALL    send
    MOVLW   'u'
    CALL    send
    MOVLW   'm'
    CALL    send
    MOVLW   'b'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   'r'
    CALL    send
    MOVLW   ' '
    CALL    send
    MOVLW   '2'
    CALL    send
    RETURN

Display_NUM2:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVLW   'N'
    CALL    send
    MOVLW   'u'
    CALL    send
    MOVLW   'm'
    CALL    send
    MOVLW   'b'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   'r'
    CALL    send
    MOVLW   ' '
    CALL    send
    MOVLW   '2'
    CALL    send
    BCF     Select, RS
    MOVLW   0xC0
    CALL    send
    BSF     Select, RS
    MOVF    NUM2_INT+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+5, W
    ADDLW   '0'
    CALL    send
    MOVLW   '.'
    CALL    send
    MOVF    NUM2_FRAC+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+5, W
    ADDLW   '0'
    CALL    send
    BCF     Select, RS
    MOVLW   0x0C
    CALL    send
    MOVLW   0x0E
    CALL    send
    MOVF    CURSOR_POS, W
    SUBLW   0x0D
    MOVWF   WORK_REG_A
    MOVF    CURSOR_POS, W
    SUBLW   5
    BTFSS   STATUS, C
    DECF    WORK_REG_A, F
Move_Cursor_NUM2:
    BCF     Select, RS
    MOVLW   0x10
    CALL    send
    DECFSZ  WORK_REG_A, F
    GOTO    Move_Cursor_NUM2
    RETURN

Show_Result:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVLW   'R'
    CALL    send
    MOVLW   'e'
    CALL    send
    MOVLW   's'
    CALL    send
    MOVLW   'u'
    CALL    send
    MOVLW   'l'
    CALL    send
    MOVLW   't'
    CALL    send
    BCF     Select, RS
    MOVLW   0xC0
    CALL    send
    BSF     Select, RS
    MOVF    CALC_RESULT+0, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+1, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+2, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+3, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+4, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+5, W
    ADDLW   '0'
    CALL    send
    MOVLW   '.'
    CALL    send
    MOVF    CALC_RESULT+6, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+7, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+8, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+9, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+0x0A, W
    ADDLW   '0'
    CALL    send
    MOVF    CALC_RESULT+0x0B, W
    ADDLW   '0'
    CALL    send
    RETURN

Show_NUM1_Final:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVF    NUM1_INT+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_INT+5, W
    ADDLW   '0'
    CALL    send
    MOVLW   '.'
    CALL    send
    MOVF    NUM1_FRAC+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM1_FRAC+5, W
    ADDLW   '0'
    CALL    send
    RETURN

Show_NUM2_Final:
    CALL    Clear_LCD
    BSF     Select, RS
    MOVF    NUM2_INT+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_INT+5, W
    ADDLW   '0'
    CALL    send
    MOVLW   '.'
    CALL    send
    MOVF    NUM2_FRAC+0, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+1, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+2, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+3, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+4, W
    ADDLW   '0'
    CALL    send
    MOVF    NUM2_FRAC+5, W
    ADDLW   '0'
    CALL    send
    RETURN

Show_Equals:
    CALL    Clear_LCD
    MOVLW   0x0C
    BCF     Select, RS
    CALL    send
    BSF     Select, RS
    MOVLW   0x3D
    CALL    send
    RETURN

;==============================================================================
; INITIALIZATION ROUTINE
;==============================================================================
Init_System:
    ; Configure PORTB
    BCF     STATUS, RP1     ; Bank 0
    BSF     STATUS, RP0     ; Bank 1
    BSF     TRISB, 0        ; RB0 as input for push button
    BSF     TRISB, 5        ; RB5 for ACK
    BSF     TRISB, 2        ; RB2 as input for co-proc interrupt
    BCF     TRISB, 1        ; RB1 as output for co-proc ACK

    ; Configure PORTD & PORTC
    CLRF    TRISD           ; PORTD as output for LCD
    BSF     TRISD, 3        ; Preserved from original
    MOVLW   0x00
    MOVWF   TRISC           ; PORTC as output initially

    ; Initialize Timer1
    BSF     STATUS, RP0     ; Bank 1
    BCF     PIE1, TMR1IE    ; Disable Timer1 interrupt
    BCF     STATUS, RP0     ; Bank 0
    MOVLW   B'00000000'
    MOVWF   T1CON
    BCF     PIR1, TMR1IF

    ; Configure Timer0
    BCF     STATUS, RP0
    MOVLW   0xD8
    MOVWF   TMR0
    CLRF    PORTD
    CLRF    PORTC

    ; Clear GPRs (0x20 to 0x59)
    MOVLW   0x20
    MOVWF   FSR
Clear_GPRs:
    CLRF    INDF
    INCF    FSR, F
    MOVLW   0x5A
    SUBWF   FSR, W
    BTFSS   STATUS, Z
    GOTO    Clear_GPRs

    ; Initialize state
    MOVLW   STATE_INIT
    MOVWF   CALC_STATE

    ; Initialize LCD
    CALL    inid
    CALL    Clear_LCD
    RETURN

;==============================================================================
; STATE HANDLER ROUTINES
;==============================================================================
Boot_Sequence:
    CALL    Clear_LCD
    CALL    Show_Welcome
    CALL    Pause_500ms
    CALL    Clear_LCD
    CALL    Show_Welcome
    CALL    Pause_500ms
    CALL    Clear_LCD
    CALL    Show_Welcome
    CALL    Pause_500ms
    RETURN

Handle_NUM1_Input:
    MOVF    CURSOR_POS, W
    MOVWF   LAST_CURSOR
    MOVLW   0xFF
    MOVWF   INPUT_TIMER
    CALL    Display_NUM1
    BSF     INTCON, INTE
    BSF     INTCON, GIE
    BSF     INTCON, PEIE
NUM1_Input_Loop:
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    BSF     INTCON, GIE
    BSF     INTCON, INTE
    MOVF    CURSOR_POS, W
    XORWF   LAST_CURSOR, W
    BTFSS   STATUS, Z
    GOTO    Check_NUM1_Cursor
    DECFSZ  INPUT_TIMER, F
    GOTO    NUM1_Input_Loop
    GOTO    Exit_NUM1_Input
Check_NUM1_Cursor:
    MOVF    CURSOR_POS, W
    XORLW   0x06
    BTFSC   STATUS, Z
    GOTO    Continue_NUM1_Loop
    GOTO    Exit_NUM1_Input
Continue_NUM1_Loop:
    DECFSZ  INPUT_TIMER, F
    GOTO    NUM1_Input_Loop
Exit_NUM1_Input:
    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF
    INCF    CURSOR_POS, F
    MOVF    CURSOR_POS, W
    SUBLW   0x0B
    BTFSC   STATUS, C
    GOTO    Handle_NUM1_Input
    RETURN

Handle_NUM2_Input:
    MOVF    CURSOR_POS, W
    MOVWF   LAST_CURSOR
    MOVLW   0xFF
    MOVWF   INPUT_TIMER
    CALL    Display_NUM2
    BSF     INTCON, INTE
    BSF     INTCON, GIE
    BSF     INTCON, PEIE
NUM2_Input_Loop:
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms
    BSF     INTCON, GIE
    BSF     INTCON, INTE
    MOVF    CURSOR_POS, W
    XORWF   LAST_CURSOR, W
    BTFSS   STATUS, Z
    GOTO    Check_NUM2_Cursor
    DECFSZ  INPUT_TIMER, F
    GOTO    NUM2_Input_Loop
    GOTO    Exit_NUM2_Input
Check_NUM2_Cursor:
    MOVF    CURSOR_POS, W
    XORLW   0x06
    BTFSC   STATUS, Z
    GOTO    Continue_NUM2_Loop
    GOTO    Exit_NUM2_Input
Continue_NUM2_Loop:
    DECFSZ  INPUT_TIMER, F
    GOTO    NUM2_Input_Loop
Exit_NUM2_Input:
    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF
    INCF    CURSOR_POS, F
    MOVF    CURSOR_POS, W
    SUBLW   0x0B
    BTFSC   STATUS, C
    GOTO    Handle_NUM2_Input
    RETURN

Switch_To_Result_Mode:
    MOVLW   STATE_DISP_RESULT
    MOVWF   CALC_STATE
    CALL    Show_Result
    CLRF    CURSOR_POS
    CLRF    LAST_CURSOR
Result_Mode_Loop:
    BSF     INTCON, INTE
    BSF     INTCON, GIE
    BSF     INTCON, PEIE
    MOVLW   STATE_DISP_RESULT
    MOVWF   CALC_STATE
    MOVF    CURSOR_POS, W
    XORWF   LAST_CURSOR, W
    BTFSS   STATUS, Z
    GOTO    Update_Result_Screen
    GOTO    Result_Mode_Loop

Update_Result_Screen:
    MOVF    CURSOR_POS, W
    MOVWF   LAST_CURSOR
    MOVF    DOUBLE_TAP_FLAG, W
    SUBLW   0x01
    BTFSC   STATUS, Z
    GOTO    Main_Event_Loop
    MOVF    CURSOR_POS, W
    BTFSC   STATUS, Z
    CALL    Show_Result
    MOVF    CURSOR_POS, W
    SUBLW   0x01
    BTFSC   STATUS, Z
    CALL    Show_NUM1_Final
    MOVF    CURSOR_POS, W
    SUBLW   0x02
    BTFSC   STATUS, Z
    CALL    Show_NUM2_Final
    GOTO    Result_Mode_Loop

;==============================================================================
; INTERRUPT SERVICE ROUTINE (ISR)
;==============================================================================
ISR_Service:
    MOVWF   ISR_W_TEMP
    SWAPF   STATUS, W
    MOVWF   ISR_STAT_TEMP

    ; Disable interrupts
    BCF     INTCON, GIE
    BCF     INTCON, INTE
    BCF     INTCON, T0IE
    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF

    ; Check result display mode
    MOVF    CALC_STATE, W
    SUBLW   STATE_DISP_RESULT
    BTFSC   STATUS, Z
    GOTO    ISR_Result_Mode

    ; Handle standard input
    CALL    Check_Double_Tap
    MOVF    DOUBLE_TAP_FLAG, W
    BTFSS   STATUS, Z
    GOTO    Handle_Double_Tap
    GOTO    Handle_Single_Tap

Set_DISP_Mode:
    MOVLW   0x01
    MOVWF   DISP_SCREEN
    RETURN

ISR_Result_Mode:
    BANKSEL CURSOR_POS
    CALL    Check_Double_Tap
    MOVF    DOUBLE_TAP_FLAG, W
    BTFSS   STATUS, Z
    CALL    Set_DISP_Mode
    
    INCF    CURSOR_POS, F
    MOVF    CURSOR_POS, W
    SUBLW   0x03
    BTFSC   STATUS, Z
    CLRF    CURSOR_POS
    GOTO    ISR_Exit

Handle_Single_Tap:
    MOVF    DISP_SCREEN, W
    BTFSC   STATUS, Z
    GOTO    Inc_Digit

    CLRF    DISP_SCREEN
    GOTO    ISR_Exit

Inc_Digit:
    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM1_INPUT
    BTFSC   STATUS, Z
    GOTO    Inc_NUM1

    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM2_INPUT
    BTFSC   STATUS, Z
    GOTO    Inc_NUM2
    GOTO    ISR_Exit

Inc_NUM1:
    MOVLW   NUM1_INT
    ADDWF   CURSOR_POS, W
    MOVWF   WORK_REG_A
    MOVWF   FSR
    MOVLW   12
    ADDWF   WORK_REG_A, W
    MOVWF   WORK_REG_A
Inc_NUM1_Loop:
    MOVF    INDF, W
    SUBLW   0x09
    BTFSC   STATUS, Z
    GOTO    Clear_NUM1_Digit
    INCF    INDF, F
    GOTO    Next_NUM1
Clear_NUM1_Digit:
    CLRF    INDF
Next_NUM1:
    INCF    FSR, F
    MOVF    FSR, W
    SUBWF   WORK_REG_A, W
    BTFSS   STATUS, Z
    GOTO    Inc_NUM1_Loop
    CALL    Display_NUM1
    GOTO    ISR_Exit

Inc_NUM2:
    MOVLW   NUM2_INT
    ADDWF   CURSOR_POS, W
    MOVWF   WORK_REG_A
    MOVWF   FSR
    MOVLW   12
    ADDWF   WORK_REG_A, W
    MOVWF   WORK_REG_A
Inc_NUM2_Loop:
    MOVF    INDF, W
    SUBLW   0x09
    BTFSC   STATUS, Z
    GOTO    Clear_NUM2_Digit
    INCF    INDF, F
    GOTO    Next_NUM2
Clear_NUM2_Digit:
    CLRF    INDF
Next_NUM2:
    INCF    FSR, F
    MOVF    FSR, W
    SUBWF   WORK_REG_A, W
    BTFSS   STATUS, Z
    GOTO    Inc_NUM2_Loop
    CALL    Display_NUM2
    GOTO    ISR_Exit

Handle_Double_Tap:
    MOVF    CURSOR_POS, W
    SUBLW   0x05
    BTFSC   STATUS, C
    GOTO    Cursor_To_Frac
    GOTO    Cursor_To_Confirm

Cursor_To_Frac:
    MOVLW   0x06
    MOVWF   CURSOR_POS
    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM1_INPUT
    BTFSC   STATUS, Z
    CALL    Display_NUM1
    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM2_INPUT
    BTFSC   STATUS, Z
    CALL    Display_NUM2
    MOVLW   0x01
    MOVWF   DISP_SCREEN
    GOTO    ISR_Exit

Cursor_To_Confirm:
    MOVLW   0x0C
    MOVWF   CURSOR_POS
    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM1_INPUT
    BTFSC   STATUS, Z
    CALL    Display_NUM1
    MOVF    CALC_STATE, W
    SUBLW   STATE_NUM2_INPUT
    BTFSC   STATUS, Z
    CALL    Display_NUM2
    GOTO    ISR_Exit

ISR_Exit:
    CALL    Pause_1ms
    CALL    Pause_1ms
    CALL    Pause_1ms

    MOVLW   0xFF
    MOVWF   INPUT_TIMER

    ; Preserved redundant block
    MOVWF   ISR_W_TEMP
    SWAPF   STATUS, W
    MOVWF   ISR_STAT_TEMP
    
    RETFIE

;==============================================================================
; MAIN PROGRAM FLOW
;==============================================================================
Main_Entry:
    CALL    Init_System
    CALL    Boot_Sequence
    CALL    Clear_LCD

Main_Event_Loop:
    CALL    Init_System
    CALL    Pause_2s
    MOVLW   STATE_NUM1_INPUT
    MOVWF   CALC_STATE
    MOVLW   0xFF
    MOVWF   INPUT_TIMER

    CALL    Handle_NUM1_Input

    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF

    MOVLW   STATE_NUM2_INPUT
    MOVWF   CALC_STATE
    CLRF    CURSOR_POS
    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF

    CALL    Reset_NUM2_Buffer
    CALL    Display_NUM2_Header
    CALL    Pause_500ms
    CALL    Pause_500ms
    MOVLW   0xFF
    MOVWF   INPUT_TIMER
    CALL    Handle_NUM2_Input

    BCF     INTCON, INTF
    BCF     INTCON, TMR0IF

    CALL    Show_Equals
    MOVLW   STATE_SEND_DATA
    MOVWF   CALC_STATE
    CALL    Transmit_To_CoProc

    CALL    Reset_Result_Buffer
    MOVLW   STATE_RECV_DATA
    MOVWF   CALC_STATE
    GOTO    Receive_From_CoProc
    GOTO    $-1

    END