Q1
INCLUDE Irvine32.inc

.data
prompt BYTE "Enter a number to multiply by 21: ", 0
resultMsg BYTE "Result of number * 21 = ", 0

.code
main PROC
    ; Print prompt
    mov edx, OFFSET prompt
    call WriteString
    
    ; Get input number from user
    call ReadInt
    ; Now EAX contains the input number
    
    ; Save original value
    mov ebx, eax        ; Save original value in EBX
    
    ; Perform multiplication by 21
    shl eax, 4         ; EAX = EAX * 16 (2^4)
    mov ecx, eax       ; Save EAX * 16 in ECX
    
    mov eax, ebx       ; Restore original value to EAX
    shl eax, 2         ; EAX = EAX * 4 (2^2)
    add eax, ecx       ; Add the *16 result
    add eax, ebx       ; Add the original value (*1)
    
    ; Print result message
    mov edx, OFFSET resultMsg
    call WriteString
    
    ; Print the result
    call WriteDec
    call Crlf
    
    exit
main ENDP
END main


Q2
INCLUDE Irvine32.inc

.data
msg1 BYTE "Initial value in AX: ", 0
msg2 BYTE "After expanding to EAX: ", 0

.code
main PROC
    ; Move -128 into AX
    mov ax, -128       ; AX = -128 (0xFF80 in hex)
    
    ; Display initial value in AX
    mov edx, OFFSET msg1
    call WriteString
    movsx eax, ax      ; Temporarily expand to show initial value
    call WriteInt
    call Crlf
    
    ; Reset back to just AX
    mov ax, -128
    
    ; Clear upper bits of EAX
    xor eax, eax
    mov ax, -128
    
    ; Expand using shifts and rotates
    ; First, preserve the sign bit
    bt ax, 15          ; Store sign bit in carry flag
    
    ; Shift left to position the bits
    shl eax, 16        ; Shift left 16 bits to move AX contents to upper word
    
    ; If it was negative, fill in the lower bits with 1's
    jnc positive       ; Jump if carry flag is 0 (positive number)
    
    ; For negative number
    or eax, 0FFFFh     ; Fill lower 16 bits with 1's
    
positive:
    ; Display final result
    mov edx, OFFSET msg2
    call WriteString
    call WriteInt
    call Crlf
    
    exit
main ENDP
END main

Q3
INCLUDE Irvine32.inc

.data
msg1 BYTE "Initial values:", 0
msg2 BYTE "AX = ", 0
msg3 BYTE "BX = ", 0
msg4 BYTE "After shift (without SHRD):", 0
msg5 BYTE "After shift (with SHRD):", 0

.code
main PROC
    ; Initialize registers
    mov ax, 1          ; AX = 0001h (lowest bit set)
    mov bx, 0F000h     ; BX = F000h (example value)
    
    ; Display initial values
    mov edx, OFFSET msg1
    call WriteString
    call Crlf
    
    mov edx, OFFSET msg2
    call WriteString
    movzx eax, ax
    call WriteHex
    call Crlf
    
    mov edx, OFFSET msg3
    call WriteString
    movzx eax, bx
    call WriteHex
    call Crlf
    call Crlf
    
    ; Save initial values for second method
    push ax
    push bx
    
    ;==========================================
    ; Method 1: Without SHRD
    ;==========================================
    
    ; Get lowest bit of AX into CF
    shr ax, 1          ; Shift right, lowest bit goes to CF
    
    ; Rotate BX right through carry
    rcr bx, 1          ; CF goes into highest bit of BX
    
    ; Display results
    mov edx, OFFSET msg4
    call WriteString
    call Crlf
    
    mov edx, OFFSET msg2
    call WriteString
    movzx eax, ax
    call WriteHex
    call Crlf
    
    mov edx, OFFSET msg3
    call WriteString
    movzx eax, bx
    call WriteHex
    call Crlf
    call Crlf
    
    ;==========================================
    ; Method 2: With SHRD
    ;==========================================
    
    ; Restore initial values
    pop bx
    pop ax
    
    ; Use SHRD instruction
    shrd bx, ax, 1     ; Shift BX right by 1, filling from AX
    shr ax, 1          ; Shift AX right by 1
    
    ; Display results
    mov edx, OFFSET msg5
    call WriteString
    call Crlf
    
    mov edx, OFFSET msg2
    call WriteString
    movzx eax, ax
    call WriteHex
    call Crlf
    
    mov edx, OFFSET msg3
    call WriteString
    movzx eax, bx
    call WriteHex
    call Crlf
    
    exit
main ENDP
END main


Q5
INCLUDE Irvine32.inc

.data
; First 64-bit number (stored as two 32-bit values)
operand1_low  DWORD 0FFFFFFFFh    ; lower 32 bits
operand1_high DWORD 1             ; upper 32 bits

; Second 64-bit number
operand2_low  DWORD 1             ; lower 32 bits
operand2_high DWORD 0             ; upper 32 bits

; Result storage
result_low    DWORD ?             ; lower 32 bits of result
result_high   DWORD ?             ; upper 32 bits of result

; Messages
msg1 BYTE "First 64-bit number:  ", 0
msg2 BYTE "Second 64-bit number: ", 0
msg3 BYTE "Sum:                  ", 0

.code
;----------------------------------------------------
Extended_Add PROC
;
; Adds two 64-bit integers.
; Receives: ESI = pointer to first operand (8 bytes)
;          EDI = pointer to second operand (8 bytes)
;          EBX = pointer to result (8 bytes)
; Returns:  nothing
;----------------------------------------------------
    push eax                    ; save registers
    push edx
    
    mov eax, [esi]             ; get low dword of first operand
    add eax, [edi]             ; add low dword of second operand
    mov [ebx], eax             ; store low dword of result
    
    mov eax, [esi + 4]         ; get high dword of first operand
    adc eax, [edi + 4]         ; add high dword of second operand with carry
    mov [ebx + 4], eax         ; store high dword of result
    
    pop edx                     ; restore registers
    pop eax
    ret
Extended_Add ENDP

;----------------------------------------------------
Display64 PROC
;
; Displays a 64-bit integer in hexadecimal.
; Receives: EDX = pointer to 64-bit integer
;----------------------------------------------------
    push eax
    
    ; Display high dword
    mov eax, [edx + 4]
    call WriteHex
    
    ; Display low dword
    mov eax, [edx]
    call WriteHex
    call Crlf
    
    pop eax
    ret
Display64 ENDP

main PROC
    ; Display first number
    mov edx, OFFSET msg1
    call WriteString
    mov edx, OFFSET operand1_low
    call Display64
    
    ; Display second number
    mov edx, OFFSET msg2
    call WriteString
    mov edx, OFFSET operand2_low
    call Display64
    
    ; Set up parameters for Extended_Add
    mov esi, OFFSET operand1_low
    mov edi, OFFSET operand2_low
    mov ebx, OFFSET result_low
    call Extended_Add
    
    ; Display result
    mov edx, OFFSET msg3
    call WriteString
    mov edx, OFFSET result_low
    call Display64
    
    exit
main ENDP
END main


Q4
INCLUDE Irvine32.inc

.data
val1 SDWORD 100   
val2 SDWORD 20
val3 SDWORD 5
temp SDWORD ?      

.code
main PROC
    ; First calculate (val2 / val3)
    mov eax, val2
    cdq             ; extend EAX sign into EDX
    mov ebx, val3
    idiv ebx       ; EAX = val2 / val3
    mov temp, eax   ; store first result
    
    ; Now calculate (val1 / val2)
    mov eax, val1
    cdq             ; extend EAX sign into EDX
    mov ebx, val2
    idiv ebx       ; EAX = val1 / val2
    
    ; Multiply the results
    imul eax, temp  ; EAX = (val2/val3) * (val1/val2)
    
    ; Store final result in val1
    mov val1, eax
    
    ; Display result
    call WriteInt
    call Crlf
    
    exit
main ENDP
END main




































![image](https://github.com/user-attachments/assets/74cbc277-ab5f-4047-86db-ea8a9c862e2c)

