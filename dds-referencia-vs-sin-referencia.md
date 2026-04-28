# DDS: Sin referencia vs. Con referencia

---

## Sin referencia — Tipo definido inline

Cada campo declara su propio tipo directamente en el fuente:

```dds
R FARTI
  ARID          6A              ← CHAR de longitud 6, definido aquí
  ARDESC       50A              ← CHAR de longitud 50, definido aquí
  ARSALEPR      7P 2            ← PACKED(7,2), definido aquí
  ARWHSPR       7P 2            ← PACKED(7,2), definido aquí (duplicado!)
  ARSTOCK       5P 0            ← PACKED(5,0), definido aquí
  ARVATCD       1A              ← CHAR(1), definido aquí
  ARDEL         1A              ← CHAR(1), definido aquí
```

**Problema**: si `ARSALEPR` necesita crecer a `9P 2`, hay que buscarlo y cambiarlo en **cada fichero** donde aparezca. Con 40+ programas y 9 PF, es un error garantizado.

---

## Con referencia — Tipo heredado de SAMREF

```dds
                                            REF(SAMREF)       ← "usa SAMREF como diccionario"
R FARTI
  ARID      R                              ← busca ARID en SAMREF → CHAR(6)
  ARDESC    R                              ← busca ARDESC en SAMREF → CHAR(50)
  ARSALEPR  R               REFFLD(UNITPRICE) ← busca UNITPRICE en SAMREF → PACKED(7,2)
  ARWHSPR   R               REFFLD(UNITPRICE) ← igual que ARSALEPR, mismo tipo
  ARSTOCK   R               REFFLD(QUANTITY)  ← busca QUANTITY en SAMREF → PACKED(5,0)
  ARVATCD   R               REFFLD(VATCODE)   ← busca VATCODE en SAMREF → CHAR(1)
  ARDEL     R               REFFLD(DLCODE)    ← busca DLCODE en SAMREF → CHAR(1)
```

---

## Comparativa directa

| Aspecto | Sin referencia | Con referencia |
|---|---|---|
| Tipo definido en | El propio PF | `SAMREF.PF` (único sitio) |
| Cambiar `UNITPRICE` de `7P2` a `9P2` | Modificar cada PF individualmente | Cambiar solo `SAMREF` y recompilar |
| Consistencia entre ficheros | Manual, propensa a errores | Garantizada |
| `REFFLD(xxx)` | No aplica | El campo puede tener un nombre distinto al de referencia |
| `R` en col. 29 | No aparece | Indica "tipo heredado" |

---

## La palabra clave `REFFLD`

Permite que el campo tenga **un nombre diferente** al del tipo en SAMREF:

```dds
  ARSALEPR  R   REFFLD(UNITPRICE)
  │         │   └─ el tipo viene de SAMREF.UNITPRICE → PACKED(7,2)
  │         └─ R = tipo heredado
  └─ nombre propio del campo en ARTICLE
```

Sin `REFFLD`, el nombre del campo debe coincidir exactamente con el de SAMREF:
```dds
  ARID      R     ← busca ARID en SAMREF (mismo nombre, no necesita REFFLD)
```

---

## Resumen

`REF(SAMREF)` + `R` es el **mecanismo de reutilización de tipos** en DDS —
equivalente a un `typedef` en C o un tipo compartido en un esquema SQL.
