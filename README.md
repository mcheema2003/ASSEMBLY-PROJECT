.ORIG x3000

    ; Initialize stack pointer
    LD   R6, STACK_PTR
    AND  R2, R2, #0    ; sum = 0
    AND  R3, R3, #0    ; count = 0

    ; Main program execution
    JSR  READ_SCORES
STACK
    JSR  PROCESS_STACK
AVERAGE
    JSR  CALCULATE_AVERAGE
    JSR  DISPLAY_RESULTS
    HALT

; Subroutine: Read Scores (Handles Multi-Digit Numbers)
READ_SCORES
    LEA  R0, PROMPT
    PUTS
    LD   R0, LF        ; Load newline character
    OUT                ; Print newline
    AND  R3, R3, #0    ; count = 0

INPUT_LOOP
    GETC               ; Read first digit
    OUT
    JSR  VALIDATE_DIGIT ; Check if valid number
    AND  R4, R4, #0
    ADD  R4, R0, #-16
    ADD  R4, R4, #-16
    ADD  R4, R4, #-16  ; Convert first digit ASCII -> integer

    LD   R5, TEN       ; Load 10 for multiplication
    JSR  MULTIPLY      ; Multiply first digit by 10 (store in R4)

    GETC               ; Read second digit
    OUT
    JSR  VALIDATE_DIGIT ; Check if valid number
    AND  R5, R5, #0
    ADD  R5, R0, #-16
    ADD  R5, R5, #-16
    ADD  R5, R5, #-16  ; Convert second digit ASCII -> integer
    LD   R0, LF        ; Load newline character
    OUT                ; Print newline

    ADD  R4, R4, R5    ; Combine first and second digits into full number

    ;push
    ADD  R6, R6, #-1   ; Move stack pointer
    STR  R4, R6, #0    ; Store full number in memory

    ADD  R3, R3, #1
    ADD  R5, R3, #-5
    BRzp EXIT_READ_SCORES ; Exit after 5 inputs
    BR   INPUT_LOOP

EXIT_READ_SCORES
    BR   STACK

; Subroutine: Validate Input (Ensures Only 0-9 is Entered)
VALIDATE_DIGIT
    AND  R1, R1, #0
    ADD  R1, R0, #-16  ; Convert ASCII to integer
    ADD  R1, R1, #-16
    ADD  R1, R1, #-16
    BRn  ERROR_INPUT   ; If less than '0', jump to error
    ADD  R1, R1, #-9   ; Check if greater than 9
    BRp  ERROR_INPUT   ; If out of range, jump to error
    RET

ERROR_INPUT
    LEA  R0, ERMS      ; Print error message
    PUTS
    HALT

; Subroutine: Multiply (R4 = R4 * R5)
MULTIPLY
    AND  R6, R6, #0    ; Initialize result (R6)
    ADD  R1, R5, #0    ; Copy multiplier to R7

MULT_LOOP
    BRz  MULT_DONE     ; If multiplier reaches 0, finish
    ADD  R6, R6, R4    ; Add R4 to result
    ADD  R1, R1, #-1   ; Decrease counter
    BR   MULT_LOOP     ; Repeat loop

MULT_DONE
    ADD  R4, R6, #0    ; Store result in R4
    RET

; Subroutine: Process Stack (Handles Multi-Digit Scores)
PROCESS_STACK
    AND  R3, R3, #0    ; Reset count
    LDR  R2, R6, #0    ; Pop first full score
    ADD  R6, R6, #1    ; Adjust stack pointer
    AND  R1, R1, #0
    ADD  R1, R2, #0    ; min = first score
    ADD  R0, R2, #0    ; max = first score
    ADD  R4, R2, #0    ; sum = first score

PROCESS_LOOP
    LDR  R5, R6, #0    ; Pop next score
    ADD  R6, R6, #1

    ; Check min
    NOT  R7, R5
    ADD  R7, R7, #1
    ADD  R7, R7, R1
    BRp  SKIP_MIN
    ADD  R1, R5, #0    ; Update min
SKIP_MIN

    ; Check max
    NOT  R7, R0
    ADD  R7, R7, #1
    ADD  R7, R7, R5
    BRn  SKIP_MAX
    ADD  R0, R5, #0    ; Update max
SKIP_MAX

    ADD  R4, R4, R5    ; Add to sum
    ADD  R3, R3, #1
    ADD  R5, R3, #-5
    BRnz PROCESS_LOOP

    ; Store values for later use
    ST   R1, MIN_STORE
    ST   R0, MAX_STORE
    ST   R4, SUM_STORE

    BR   AVERAGE

; Subroutine: Calculate Average (Handles Multi-Digit Numbers)
CALCULATE_AVERAGE
    LD   R5, FIVE      ; Load divisor (5)
    AND  R6, R6, #0    ; Reset counter
AVG_LOOP
    LD   R4, SUM_STORE
    ADD  R4, R4, #-5   ; Subtract 5 from sum
    BRn  AVG_DONE      ; Stop when sum < 5
    ADD  R6, R6, #1    ; Increment count (acts as division)
    BR   AVG_LOOP

AVG_DONE
    ST   R6, AVG_STORE ; Store average for later use
    RET

; Subroutine: Display Results
DISPLAY_RESULTS
    LEA  R0, MSG_MIN
    PUTS
    LD   R0, MIN_STORE ; Load stored min value
    JSR  PRINT_NUMBER  ; Print min value
    LD   R0, MIN_STORE ; Load stored min value again
    JSR  LETTER_GRADE  ; Display corresponding letter grade

    LEA  R0, MSG_MAX
    PUTS
    LD   R0, MAX_STORE ; Load stored max value
    JSR  PRINT_NUMBER  ; Print max value
    LD   R0, MAX_STORE ; Load stored max value again
    JSR  LETTER_GRADE  ; Display corresponding letter grade

    LEA  R0, MSG_AVG
    PUTS
    LD   R0, AVG_STORE ; Load stored average value
    JSR  PRINT_NUMBER  ; Print average value
    LD   R0, AVG_STORE ; Load stored average value again
    JSR  LETTER_GRADE  ; Display corresponding letter grade

    RET

; Subroutine: Print 2-Digit Numbers
PRINT_NUMBER
    AND  R5, R5, #0    ; Clear register
    LD   R5, TEN       ; Load 10 for division
    AND  R6, R6, #0    ; Clear temporary storage
    AND  R4, R4, #0    ; Clear remainder storage

    ; Calculate tens place
    ADD  R6, R0, #0    ; Copy number to R6
TEN_LOOP
    ADD  R6, R6, #-10  ; Subtract 10
    BRn  DONE_TENS
    ADD  R4, R4, #1    ; Increment tens count
    BR   TEN_LOOP
DONE_TENS
    ADD  R6, R6, #10   ; Restore remainder

    ; Convert tens place to ASCII
    ADD  R4, R4, #15
    ADD  R4, R4, #15
    ADD  R4, R4, #15
    ADD  R4, R4, #3
    ADD  R1, R0, #0
    ADD  R0, R4, #0
    OUT
    ADD  R0, R1, #0

    ; Convert ones place to ASCII
    ADD  R6, R6, #15
    ADD  R6, R6, #15
    ADD  R6, R6, #15
    ADD  R6, R6, #3
    ADD  R1, R0, #0
    ADD  R0, R6, #0
    OUT
    ADD  R0, R1, #0
    RET

; Subroutine: Letter Grade (Working Like Days)
LETTER_GRADE
    LEA  R0, LETTERS   ; Load letter grade array
    ADD  R4, R0, #0    ; Set index

LETTER_LOOP
    BRz  DISPLAY_GRADE ; If index is 0, display grade
    ADD  R0, R0, #2    ; Move to next grade (2 bytes each)
    ADD  R4, R4, #-1   ; Decrement index
    BR   LETTER_LOOP   ; Repeat loop

DISPLAY_GRADE
    PUTS
    RET

; Data Section
PROMPT     .STRINGZ "Enter 5 exam scores: "
MSG_MIN    .STRINGZ "\nMinimum: "
MSG_MAX    .STRINGZ "\nMaximum: "
MSG_AVG    .STRINGZ "\nAverage: "

MIN_STORE  .FILL 0   ; Stores minimum value
MAX_STORE  .FILL 0   ; Stores maximum value
SUM_STORE  .FILL 0   ; Stores sum of scores
AVG_STORE  .FILL 0   ; Stores average value

LETTERS    .STRINGZ "F " ; 0 ? F
           .STRINGZ "D " ; 1 ? D
           .STRINGZ "C " ; 2 ? C
           .STRINGZ "B " ; 3 ? B
           .STRINGZ "A " ; 4 ? A

ERMS       .STRINGZ "Invalid input!"

STACK_PTR  .FILL xFDFF
NEG_FIVE   .FILL #-5
FIVE       .FILL #5  ; Used for division
TEN        .FILL #10 ; Used for multiplication (changed xA to #10 for consistency)
LF         .FILL x000A

.END
