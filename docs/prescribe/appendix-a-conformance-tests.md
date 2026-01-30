## Appendix A. Conformance Tests

These are minimal, must-pass programs for a compliant implementation.

**Expressions**
```prescribe
PROGRAM ExprTest
    DECLARE A : INTEGER
    A <- 7 DIV 3
    OUTPUT A
ENDPROGRAM
```

**Control flow**
```prescribe
PROGRAM LoopTest
    DECLARE I : INTEGER
    FOR I <- 1 TO 3
        OUTPUT I
    NEXT I
ENDPROGRAM
```

**Procedures and functions**
```prescribe
PROGRAM CallTest
    OUTPUT Inc(5)
ENDPROGRAM

FUNCTION Inc(X : INTEGER) RETURNS INTEGER
    RETURN X + 1
ENDFUNCTION
```

**Files**
```prescribe
PROGRAM FileTest
    DECLARE F : TEXTFILE
    OPENFILE(F, "t.txt", "WRITE")
    WRITEFILE(F, "OK")
    CLOSEFILE(F)
ENDPROGRAM
```

**OOP**
```prescribe
PROGRAM OopTest
    DECLARE P : Person
    P <- NEW Person("Ada")
    OUTPUT P.GetName()
ENDPROGRAM

CLASS Person
    PRIVATE
        Name : STRING
    PUBLIC
        CONSTRUCTOR Person(NewName : STRING)
            Name <- NewName
        ENDCONSTRUCTOR

        FUNCTION GetName() RETURNS STRING
            RETURN Name
        ENDFUNCTION
ENDCLASS
```

---

