INCLUDE Irvine32.inc

.data
val1 DWORD ?
val2 DWORD ?
val3 DWORD ?

.code
Calculate PROC
    push ebp
    mov ebp, esp

    ; Compute val2 / val3
    mov eax, val2
    cdq
    idiv val3
    push eax   ; Save quotient of val2 / val3

    ; Compute val1 / val2
    mov eax, val1
    cdq
    idiv val2

    ; Multiply the two quotients
    pop ebx    ; EBX holds val2 / val3
    imul ebx

    ; Assign result to val1
    mov val1, eax

    pop ebp
    ret
Calculate ENDP

main PROC
    mov val1, 100
    mov val2, 20
    mov val3, 5

    call Calculate
    call writeint
    call ExitProcess
main ENDP
END main













![image](https://github.com/user-attachments/assets/74cbc277-ab5f-4047-86db-ea8a9c862e2c)

