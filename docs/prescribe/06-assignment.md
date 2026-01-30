## 6. Assignment

Assignment uses `<-` (ASCII) or `←` (Unicode). Both are equivalent.

Syntax:
```prescribe
<lvalue> <- <expression>
```

Valid lvalues:
- Variable
- Array element
- Record field
- Dereferenced pointer

Type matching rules:
- The expression type must exactly match the lvalue type.
- No implicit conversions.

---

