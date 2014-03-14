#C++ Small String Optimize
Most STL implement use a technical call Small String Optimize(SSO), that store data direct in the string object instead of alloc storage data from heap, which will run faster(no malloc/free needed). Follow is proof of concept how does it implement:

    class string {
    public:
        // all 83 member functions
    private:
        size_type m_size;
        union {
            class {
                // This is probably better designed as an array-like class
                std::unique_ptr<char[]> m_data;
                size_type m_capacity;
            } m_large;
            std::array<char, sizeof(m_large)> m_small;
        };
    };
    
and this is how it look in assembly
    
    .text:00A21074 030 A1 18 30 A2 00                  mov     eax, ___security_cookie
    .text:00A21079 030 33 C5                           xor     eax, ebp                ; Logical Exclusive OR
	.text:00A2107B 030 89 45 F0                        mov     [ebp+var_10], eax
	.text:00A2107E 030 56                              push    esi
	.text:00A2107F 034 57                              push    edi
	.text:00A21080 038 50                              push    eax
	.text:00A21081 03C 8D 45 F4                        lea     eax, [ebp+var_C]        ; Load Effective Address
	.text:00A21084 03C 64 A3 00 00 00 00               mov     large fs:0, eax
	.text:00A2108A 03C C7 45 E8 0F 00 00 00            mov     [ebp+inp.baseclass_0._Myres], 0Fh
	.text:00A21091 03C C7 45 E4 00 00 00 00            mov     [ebp+inp.baseclass_0._Mysize], 0
	.text:00A21098 03C C6 45 D4 00                     mov     byte ptr [ebp+inp.baseclass_0.___u0], 0
	.text:00A2109C 03C C7 45 FC 00 00 00 00            mov     [ebp+var_4], 0
	.text:00A210A3 03C A1 6C 20 A2 00                  mov     eax, ds:std::basic_istream<char,std::char_traits<char>> std::cin
	.text:00A210A8 03C 8B 08                           mov     ecx, [eax]
	.text:00A210AA 03C 8B 49 04                        mov     ecx, [ecx+4]
	.text:00A210AD 03C 6A 0A                           push    0Ah
	.text:00A210AF 040 03 C8                           add     ecx, eax                ; Add
	.text:00A210B1 040 8B F0                           mov     esi, eax
	.text:00A210B3 040 FF 15 64 20 A2 00               call    ds:std::basic_ios<char,std::char_traits<char>>::widen(char) ; Indirect Call Near Procedure
	.text:00A210B3
	.text:00A210B9 038 0F B6 D0                        movzx   edx, al                 ; Move with Zero-Extend
	.text:00A210BC 038 52                              push    edx                     ; _Delim
	.text:00A210BD 03C 56                              push    esi                     ; _Istr
	.text:00A210BE 040 8D 4D D4                        lea     ecx, [ebp+inp]          ; _Str
	.text:00A210C1 040 E8 7A 03 00 00                  call    ??$getline@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@YAAAV?$basic_istream@DU?$char_traits@D@std@@@0@$$QAV10@AAV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@0@D@Z ; Call Procedure
	.text:00A210C1
	.text:00A210C6 040 8B 7D D4                        mov     edi, dword ptr [ebp+inp.baseclass_0.___u0]
	.text:00A210C9 040 83 C4 08                        add     esp, 8                  ; Add
	.text:00A210CC 038 83 7D E8 10                     cmp     [ebp+inp.baseclass_0._Myres], 10h ; Compare Two Operands
	.text:00A210D0 038 73 03                           jnb     short loc_A210D5        ; Jump if Not Below (CF=0)
	.text:00A210D0
	.text:00A210D2 038 8D 7D D4                        lea     edi, [ebp+inp]          ; Load Effective Address
	.text:00A210D2
	.text:00A210D5
	.text:00A210D5                             loc_A210D5:                             ; CODE XREF: _main+70j
	.text:00A210D5 038 8B 75 E4                        mov     esi, [ebp+inp.baseclass_0._Mysize]
	.text:00A210D8 038 33 C0                           xor     eax, eax                ; Logical Exclusive OR
	.text:00A210DA 038 BA 01 00 00 00                  mov     edx, 1
	.text:00A210DF 038 85 F6                           test    esi, esi                ; Logical Compare
	.text:00A210E1 038 74 12                           jz      short loc_A210F5        ; Jump if Zero (ZF=1)
	.text:00A210E1
	.text:00A210E3
	.text:00A210E3                             loc_A210E3:                             ; CODE XREF: _main+93j
	.text:00A210E3 038 8B C8                           mov     ecx, eax
	.text:00A210E5 038 83 E1 1F                        and     ecx, 1Fh                ; Logical AND
	.text:00A210E8 038 D3 E2                           shl     edx, cl                 ; Shift Logical Left
	.text:00A210EA 038 0F B6 0C 07                     movzx   ecx, byte ptr [edi+eax] ; Move with Zero-Extend
	.text:00A210EE 038 40                              inc     eax                     ; Increment by 1
	.text:00A210EF 038 03 D1                           add     edx, ecx                ; Add
	.text:00A210F1 038 3B C6                           cmp     eax, esi                ; Compare Two Operands
	.text:00A210F3 038 72 EE                           jb      short loc_A210E3        ; Jump if Below (CF=1)
	.text:00A210F3
	.text:00A210F5
	.text:00A210F5                             loc_A210F5:                             ; CODE XREF: _main+81j
	.text:00A210F5 038 A1 44 20 A2 00                  mov     eax, ds:std::endl(std::basic_ostream<char,std::char_traits<char>> &)
	.text:00A210FA 038 8B 0D 70 20 A2 00               mov     ecx, ds:std::basic_ostream<char,std::char_traits<char>> std::cout
	.text:00A21100 038 50                              push    eax
	.text:00A21101 03C 52                              push    edx
	.text:00A21102 040 FF 15 58 20 A2 00               call    ds:std::basic_ostream<char,std::char_traits<char>>::operator<<(uint) ; Indirect Call Near Procedure
	.text:00A21102
	.text:00A21108 038 8B C8                           mov     ecx, eax
	.text:00A2110A 038 FF 15 54 20 A2 00               call    ds:std::basic_ostream<char,std::char_traits<char>>::operator<<(std::basic_ostream<char,std::char_traits<char>> & (*)(std::basic_ostream<char,std::char_traits<char>> &)) ; Indirect Call Near Procedure
	.text:00A2110A
	.text:00A21110 038 83 7D E8 10                     cmp     [ebp+inp.baseclass_0._Myres], 10h ; Compare Two Operands
	.text:00A21114 038 72 0D                           jb      short loc_A21123        ; Jump if Below (CF=1)
	.text:00A21114
	.text:00A21116 038 8B 4D D4                        mov     ecx, dword ptr [ebp+inp.baseclass_0.___u0]
	.text:00A21119 038 51                              push    ecx
	.text:00A2111A 03C FF 15 D8 20 A2 00               call    ds:operator delete(void *) ; Indirect Call Near Procedure
	.text:00A2111A
	.text:00A21120 03C 83 C4 04                        add     esp, 4                  ; Add