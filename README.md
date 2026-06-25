# Lunaria Stock S.A.

Sistema academico de ventas, compras e inventario hecho con Django.

Esta guia es para practicar la leccion de ORM desde la consola de Django. Esta escrita como apoyo directo: que escribir, que significa lo que sale y que decir si el profesor pregunta.

## Idea Principal

- El ORM permite trabajar con la base de datos usando clases de Python.
- Una factura es una venta: cuando se emite, baja el stock.
- Una compra es entrada de mercaderia: cuando se confirma, sube el stock.
- Un documento en borrador se puede editar o eliminar.
- Un documento ya emitido o confirmado no se borra; se anula para conservar historial.
- Los detalles calculan subtotal e IVA automaticamente cuando se guardan.
- La pagina y la consola usan los mismos modelos.

## Modelos Clave

| App | Modelos |
|-----|---------|
| `billing` | `Brand`, `ProductGroup`, `Supplier`, `Product`, `Customer`, `CustomerProfile`, `Invoice`, `InvoiceDetail`, `CreditNote` |
| `purchasing` | `Purchase`, `PurchaseDetail`, `SupplierCreditNote` |
| `inventory` | `StockMovement` |

Relaciones que debo explicar:

| Relacion | Ejemplo | Que digo |
|----------|---------|----------|
| `ForeignKey` | `Product.brand`, `Invoice.customer` | Muchos registros pertenecen a uno. |
| `ManyToMany` | `Product.suppliers` | Un producto puede tener varios proveedores. |
| `OneToOne` | `Customer.profile` | Un cliente tiene un solo perfil. |

## Preparar El Proyecto

En PowerShell entro a la carpeta:

```powershell
cd "C:\Users\Brithany Gines\Downloads\SalesProjectDjango-main"
```

Activo el entorno:

```powershell
.\venv\Scripts\activate
```

Reviso que todo este bien:

```powershell
python manage.py check
```

Debe salir:

```text
System check identified no issues (0 silenced).
```

Si no reconoce paquetes, uso:

```powershell
.\venv\Scripts\python.exe manage.py check
```

Para abrir la pagina:

```powershell
python manage.py runserver
```

URL:

```text
http://127.0.0.1:8000/
```

## Entrar A La Consola ORM

Si estoy en PowerShell:

```text
(venv) PS C:\Users\Brithany Gines\Downloads\SalesProjectDjango-main>
```

entro a Django con:

```powershell
python manage.py shell
```

Puede salir asi:

```text
>>>
```

o asi:

```text
In [1]:
```

Ambos estan bien. `In [1]:` significa que estoy usando IPython.

Regla importante:

```text
(venv) PS ...>   = PowerShell. Aqui escribo python manage.py shell.
In [1]:          = Django/Python. Aqui escribo imports y ORM.
>>>              = Django/Python. Aqui escribo imports y ORM.
```

Si escribo `exit()`, salgo de Django y vuelvo a PowerShell. Para seguir con ORM debo entrar otra vez:

```powershell
python manage.py shell
```

## Imports Obligatorios

Cuando ya estoy en `In [1]:` o `>>>`, pego esto:

```python
from decimal import Decimal
from django.db import transaction
from django.db.models import F, Q, Sum, Avg, Max, Min, Count
from billing.models import Brand, ProductGroup, Supplier, Product, Customer, CustomerProfile, Invoice, InvoiceDetail, CreditNote
from purchasing.models import Purchase, PurchaseDetail, SupplierCreditNote
from inventory.models import StockMovement
```

Si no sale error, digo:

```text
Ya importe los modelos. Ahora puedo trabajar con las tablas usando el ORM de Django.
```

## Errores Reales Que Ya Me Pasaron

### 1. Escribi `Brand.objects.all` sin parentesis

Si escribo:

```python
Brand.objects.all
```

sale algo como:

```text
<bound method BaseManager.all ...>
```

No es error grave. Significa que mostre la funcion, pero no la ejecute.

Lo correcto es:

```python
Brand.objects.all()
```

Digo:

```text
Me faltaron los parentesis. En ORM, all() ejecuta la consulta.
```

### 2. Sali con `exit()` y pegue imports en PowerShell

Si veo:

```text
(venv) PS ...>
```

no debo pegar:

```python
from decimal import Decimal
```

Primero entro otra vez:

```powershell
python manage.py shell
```

Digo:

```text
Los imports son de Python, por eso deben pegarse dentro de la consola de Django, no en PowerShell.
```

### 3. Me salio `<QuerySet []>`

Ejemplo:

```python
Product.objects.filter(name__icontains='laptop')
```

Si sale:

```text
<QuerySet []>
```

no es error. Significa que no encontro productos con esa palabra.

Pruebo con una palabra que si exista:

```python
Product.objects.filter(name__icontains='lenovo')
Product.objects.filter(name__icontains='mouse')
Product.objects.filter(name__icontains='teclado')
```

Digo:

```text
El QuerySet vacio significa que la consulta esta bien escrita, pero no hay datos que coincidan.
```

## Guion Rapido Para La Leccion

### Si me pregunta: "Muestrame las marcas"

Escribo:

```python
Brand.objects.all()
```

Digo:

```text
Estoy consultando todos los registros de la tabla de marcas. El resultado es un QuerySet.
```

### Si me pregunta: "Muestrame los productos"

Escribo:

```python
Product.objects.all()
```

Digo:

```text
Estoy consultando todos los productos. Cada producto esta relacionado con una marca y un grupo.
```

### Si me pregunta: "Busca un producto"

Escribo:

```python
Product.objects.filter(name__icontains='lenovo')
```

Digo:

```text
Uso filter porque puede devolver varios resultados. icontains busca texto sin importar mayusculas.
```

### Si me pregunta: "Crea marca, grupo, proveedor y producto"

Escribo:

```python
marca = Brand.objects.create(name='Bri Marca Leccion', description='Marca creada desde consola')
grupo = ProductGroup.objects.create(name='Bri Grupo Leccion')
proveedor = Supplier.objects.create(name='Bri Proveedor Leccion', email='proveedor.leccion@bri.ec')
producto = Product.objects.create(
    name='Bri Producto Leccion',
    description='Producto creado en la leccion',
    brand=marca,
    group=grupo,
    unit_price=Decimal('50.00'),
    stock=20
)
producto.suppliers.add(proveedor)
producto.suppliers.all()
```

Digo:

```text
Cree registros con objects.create. El producto usa ForeignKey para marca y grupo. Luego use add() para relacionarlo con proveedor usando ManyToMany.
```

Si sale `UNIQUE constraint failed`, cambio el nombre:

```python
marca = Brand.objects.create(name='Bri Marca Leccion 2')
grupo = ProductGroup.objects.create(name='Bri Grupo Leccion 2')
```

### Si me pregunta: "Actualiza el stock"

Escribo:

```python
Product.objects.filter(pk=producto.pk).update(stock=F('stock') + 5)
producto.refresh_from_db()
producto.stock
```

Digo:

```text
Use F('stock') para sumar usando el valor actual de la base de datos. refresh_from_db actualiza mi variable.
```

### Si me pregunta: "Crea un cliente y su perfil"

Escribo:

```python
cliente = Customer.objects.create(
    dni='0926687856',
    first_name='Brithany',
    last_name='Gines',
    email='brithany.leccion@example.com'
)
perfil = CustomerProfile.objects.create(
    customer=cliente,
    taxpayer_type='ruc',
    payment_terms='credit_30',
    credit_limit=Decimal('800.00')
)
cliente.profile
```

Digo:

```text
Cree un cliente y su perfil. Esta es una relacion OneToOne porque cada cliente tiene un solo perfil.
```

Si la cedula ya existe:

```python
cliente = Customer.objects.create(
    dni='1710034065',
    first_name='Brithany',
    last_name='Gines',
    email='brithany.leccion2@example.com'
)
```

### Si me pregunta: "Crea una factura"

Escribo:

```python
factura = Invoice.objects.create(customer=cliente, estado=Invoice.BORRADOR)
detalle = InvoiceDetail.objects.create(
    invoice=factura,
    product=producto,
    quantity=2,
    unit_price=producto.unit_price,
    discount_pct=Decimal('0.00')
)
factura.subtotal = detalle.subtotal
factura.tax = detalle.tax_amount
factura.total = factura.subtotal + factura.tax
factura.save()
factura.get_estado_display()
```

Digo:

```text
Cree la cabecera de la factura y luego el detalle. El detalle calcula subtotal e IVA automaticamente. La factura queda en Borrador.
```

### Si me pregunta: "Agrega otro producto a la factura"

Escribo:

```python
producto2 = Product.objects.create(
    name='Bri Cargador Leccion',
    description='Producto adicional',
    brand=marca,
    group=grupo,
    unit_price=Decimal('25.00'),
    stock=10
)
detalle2 = InvoiceDetail.objects.create(
    invoice=factura,
    product=producto2,
    quantity=1,
    unit_price=producto2.unit_price,
    discount_pct=Decimal('0.00')
)
totales = factura.details.aggregate(subtotal=Sum('subtotal'), tax=Sum('tax_amount'))
factura.subtotal = totales['subtotal'] or Decimal('0.00')
factura.tax = totales['tax'] or Decimal('0.00')
factura.total = factura.subtotal + factura.tax
factura.save()
factura.total
```

Digo:

```text
Agregue otro detalle a la misma factura y recalcule los totales sumando todos los detalles.
```

### Si me pregunta: "Quita un producto de la factura"

Solo si esta en borrador:

```python
factura.get_estado_display()
detalle2.delete()
totales = factura.details.aggregate(subtotal=Sum('subtotal'), tax=Sum('tax_amount'))
factura.subtotal = totales['subtotal'] or Decimal('0.00')
factura.tax = totales['tax'] or Decimal('0.00')
factura.total = factura.subtotal + factura.tax
factura.save()
```

Digo:

```text
Puedo quitar detalles mientras la factura esta en Borrador. Si ya esta emitida, no debo modificarla directamente.
```

### Si me pregunta: "Emite la factura"

Primero reviso stock:

```python
for d in factura.details.select_related('product'):
    print(d.product.name, d.product.stock, 'disponible /', d.quantity, 'requerido')
```

Luego emito:

```python
with transaction.atomic():
    for d in factura.details.select_related('product'):
        Product.objects.filter(pk=d.product_id).update(stock=F('stock') - d.quantity)
        StockMovement.objects.create(
            product=d.product,
            quantity=-d.quantity,
            movement_type=StockMovement.VENTA,
            invoice=factura,
            notes='Venta desde consola ORM'
        )
    factura.estado = Invoice.EMITIDA
    factura.save()
```

Compruebo:

```python
factura.refresh_from_db()
factura.get_estado_display()
Product.objects.filter(pk=producto.pk).first().stock
StockMovement.objects.filter(invoice=factura)
```

Digo:

```text
Al emitir, la factura cambia de Borrador a Emitida. Como es una venta, baja el stock y se registra un movimiento tipo VENTA.
```

### Si me pregunta: "Anula la factura"

Escribo:

```python
with transaction.atomic():
    for d in factura.details.select_related('product'):
        Product.objects.filter(pk=d.product_id).update(stock=F('stock') + d.quantity)
        StockMovement.objects.create(
            product=d.product,
            quantity=d.quantity,
            movement_type=StockMovement.DEVOLUCION_VENTA,
            invoice=factura,
            notes='Anulacion desde consola ORM'
        )
    factura.estado = Invoice.ANULADA
    factura.is_active = False
    factura.save()
```

Compruebo:

```python
factura.refresh_from_db()
factura.get_estado_display()
```

Digo:

```text
No borro una factura emitida. La anulo para conservar historial y devuelvo el stock.
```

### Si me pregunta: "Crea y confirma una compra"

Escribo:

```python
compra = Purchase.objects.create(supplier=proveedor, document_number='BRI-LEC-001')
compra_detalle = PurchaseDetail.objects.create(
    purchase=compra,
    product=producto,
    quantity=10,
    unit_cost=Decimal('30.00')
)
compra.subtotal = compra_detalle.subtotal
compra.tax = compra_detalle.tax_amount
compra.total = compra.subtotal + compra.tax
compra.save()
```

Confirmo:

```python
with transaction.atomic():
    for d in compra.details.select_related('product'):
        Product.objects.filter(pk=d.product_id).update(stock=F('stock') + d.quantity)
        StockMovement.objects.create(
            product=d.product,
            quantity=d.quantity,
            movement_type=StockMovement.COMPRA,
            purchase=compra,
            notes='Compra desde consola ORM'
        )
    compra.estado = Purchase.CONFIRMADA
    compra.save()
```

Compruebo:

```python
compra.refresh_from_db()
compra.get_estado_display()
Product.objects.filter(pk=producto.pk).first().stock
```

Digo:

```text
La compra queda Confirmada. Como representa entrada de mercaderia, aumenta el stock.
```

### Si me pregunta: "Anula una compra"

Escribo:

```python
with transaction.atomic():
    for d in compra.details.select_related('product'):
        Product.objects.filter(pk=d.product_id).update(stock=F('stock') - d.quantity)
        StockMovement.objects.create(
            product=d.product,
            quantity=-d.quantity,
            movement_type=StockMovement.DEVOLUCION_COMPRA,
            purchase=compra,
            notes='Anulacion de compra desde consola ORM'
        )
    compra.estado = Purchase.ANULADA
    compra.is_active = False
    compra.save()
```

Digo:

```text
Al anular una compra confirmada, se revierte el stock porque esa entrada de mercaderia ya no cuenta.
```

### Si me pregunta: "Elimina algo"

Uso un dato temporal:

```python
temporal = Brand.objects.create(name='Bri Temporal Delete')
temporal.delete()
```

Digo:

```text
Esto demuestra DELETE con un dato de prueba. No elimino facturas emitidas ni compras confirmadas; esas se anulan.
```

## Consultas Utiles

Filtrar:

```python
Product.objects.filter(unit_price__gt=250)
Product.objects.filter(stock__lte=5, is_active=True)
Invoice.objects.filter(estado=Invoice.EMITIDA)
Purchase.objects.filter(estado=Purchase.CONFIRMADA)
```

Relaciones:

```python
producto.brand.name
marca.products.all()
producto.suppliers.all()
proveedor.products.all()
cliente.profile.credit_limit
perfil.customer.full_name
```

Agregaciones:

```python
Product.objects.aggregate(promedio=Avg('unit_price'))
Product.objects.aggregate(maximo=Max('unit_price'), minimo=Min('unit_price'))
Invoice.objects.filter(estado=Invoice.EMITIDA).aggregate(total=Sum('total'))
Brand.objects.annotate(cantidad=Count('products')).values('name', 'cantidad')
```

Consulta con `Q`:

```python
Product.objects.filter(Q(brand__name='Samsung') | Q(unit_price__gt=1000))
```

Digo:

```text
Q permite hacer condiciones OR. aggregate resume datos generales y annotate calcula por cada registro.
```

## Notas De Credito

Nota de credito de factura:

```python
factura_emitida = Invoice.objects.filter(estado=Invoice.EMITIDA).first()
nota = CreditNote.objects.create(
    invoice=factura_emitida,
    tipo=CreditNote.TIPO_TOTAL,
    amount=factura_emitida.total,
    reason='Devolucion total de la venta'
)
```

Nota de credito a proveedor:

```python
compra_confirmada = Purchase.objects.filter(estado=Purchase.CONFIRMADA).first()
nota_proveedor = SupplierCreditNote.objects.create(
    purchase=compra_confirmada,
    tipo=SupplierCreditNote.TIPO_TOTAL,
    amount=compra_confirmada.total,
    reason='Devolucion total al proveedor'
)
```

Digo:

```text
La nota de credito queda vinculada por ForeignKey al documento original.
```

## Errores Comunes

| Error | Que significa | Solucion |
|-------|---------------|----------|
| `NameError` | No importe el modelo o sali de la consola | Pegar imports dentro de `In [1]:` o `>>>` |
| `UNIQUE constraint failed` | El dato unico ya existe | Cambiar nombre, cedula o documento |
| `DoesNotExist` | Busque algo exacto que no existe | Revisar con `.all()` o usar `.filter()` |
| `ProtectedError` | Quiero borrar algo relacionado | No borrar; anular o usar dato temporal |
| `QuerySet []` | No hay resultados | Cambiar el filtro por un dato que exista |
| `from no se admite` | Pegue Python en PowerShell | Ejecutar `python manage.py shell` |

Cedulas validas para practicar:

```text
0926687856
1710034065
```

## Equivalencia Pagina Y Consola

| Accion | Pagina | Consola |
|--------|--------|---------|
| Crear factura | `Facturas` -> `Nueva Factura` | `Invoice` + `InvoiceDetail` |
| Emitir factura | Boton `Emitir` | Estado `EMITIDA` y resta stock |
| Anular factura | Boton `Anular` | Estado `ANULADA` y devuelve stock |
| Crear compra | `Compras` -> `Nueva Compra` | `Purchase` + `PurchaseDetail` |
| Confirmar compra | Boton `Confirmar` | Estado `CONFIRMADA` y suma stock |
| Anular compra | Boton `Anular` | Estado `ANULADA` y revierte stock |

## Respuesta Para Memorizar

```text
El proyecto usa Django ORM para manejar ventas, compras e inventario.
En ventas creo una factura en borrador, agrego detalles y calculo subtotal, IVA y total.
Cuando emito la factura, baja el stock y registro un movimiento de inventario.
Si ya fue emitida, no la elimino; la anulo y devuelvo el stock.

En compras creo una compra en borrador, agrego detalles y calculo totales.
Cuando confirmo la compra, sube el stock y registro un movimiento de inventario.
Si se anula una compra confirmada, se revierte el stock.

La diferencia clave es: factura baja inventario; compra sube inventario.
```

## Equipo

| Nombre | Rol |
|--------|-----|
| Vera Paredes Daniel | Profesor |
| Delgado Zambrano Alexy | 4to semestre |
| Gines Moncada Brithany | 4to semestre |
| Lopez Herrera Ashley | 4to semestre |
| Martinez Lopez Byron | 4to semestre |
| Moreira Intriago Diego | 4to semestre |
| Quizhpi Landi Andy | 4to semestre |
