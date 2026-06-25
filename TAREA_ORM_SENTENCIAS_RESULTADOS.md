# Tarea ORM - Sentencias y Resultados

Proyecto: Lunaria Stock S.A.  
Objetivo: Practicar operaciones CRUD y consultas desde la consola de Django, sin usar la página web.

La lección se trabaja desde la consola porque ahí se ve directo cómo el ORM conversa con la base de datos. De todas formas, el programa también permite hacerlo desde la página: los botones de facturas y compras usan los mismos modelos que se usan aquí con sentencias.

## Preparación

**Sentencia**
```bash
python manage.py shell
```

**Resultado**
```text
25 objects imported automatically (use -v 2 for details).
>>>
```

**Sentencia**
```python
from decimal import Decimal
from django.db.models import F, Q, Sum, Avg, Max, Min, Count
from billing.models import Brand, ProductGroup, Supplier, Product, Customer, CustomerProfile, Invoice, InvoiceDetail, CreditNote
from purchasing.models import Purchase, PurchaseDetail, SupplierCreditNote
```

**Resultado**
```text
>>>
```

---

## 10.1 CREATE

### Crear una marca

**Sentencia**
```python
infinix = Brand.objects.create(name='Infinix', description='Electronics')
```

**Resultado**
```text
INSERT INTO "billing_brand" ("name", "description", "is_active", "created_at", "updated_at")
VALUES ('Infinix', 'Electronics', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_brand"."id";
```

### Crear otra marca

**Sentencia**
```python
xiaomi = Brand.objects.create(name='Xiaomi', description='Smart devices')
```

**Resultado**
```text
INSERT INTO "billing_brand" ("name", "description", "is_active", "created_at", "updated_at")
VALUES ('Xiaomi', 'Smart devices', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_brand"."id";
```

### Crear grupo de productos

**Sentencia**
```python
phones = ProductGroup.objects.create(name='Celulares Premium')
```

**Resultado**
```text
INSERT INTO "billing_productgroup" ("name", "is_active", "created_at", "updated_at")
VALUES ('Celulares Premium', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_productgroup"."id";
```

### Crear proveedor principal

**Sentencia**
```python
aurora = Supplier.objects.create(name='Aurora Distribuciones', email='ventas@aurora.ec', phone='0991112223')
```

**Resultado**
```text
INSERT INTO "billing_supplier" ("name", "contact_name", "email", "phone", "address", "photo", "is_active", "created_at", "updated_at")
VALUES ('Aurora Distribuciones', NULL, 'ventas@aurora.ec', '0991112223', NULL, '', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_supplier"."id";
```

### Crear segundo proveedor

**Sentencia**
```python
elite = Supplier.objects.create(name='Elite Mobile Supply', email='contacto@elite.ec')
```

**Resultado**
```text
INSERT INTO "billing_supplier" ("name", "contact_name", "email", "phone", "address", "photo", "is_active", "created_at", "updated_at")
VALUES ('Elite Mobile Supply', NULL, 'contacto@elite.ec', NULL, NULL, '', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_supplier"."id";
```

### Crear producto

**Sentencia**
```python
note = Product.objects.create(
    name='Infinix Note 40',
    description='Teléfono de gama media con carga rápida',
    brand=infinix,
    group=phones,
    unit_price=Decimal('289.99'),
    stock=35
)
```

**Resultado**
```text
INSERT INTO "billing_product" ("name", "description", "brand_id", "group_id", "photo", "unit_price", "tax_rate", "stock", "is_active", "created_at", "updated_at")
VALUES ('Infinix Note 40', 'Teléfono de gama media con carga rápida', 1, 1, '', '289.99', '0.1500', 35, 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_product"."id";
```

### Agregar proveedores al producto ManyToMany

**Sentencia**
```python
note.suppliers.add(aurora, elite)
```

**Resultado**
```text
INSERT OR IGNORE INTO "billing_product_suppliers" ("product_id", "supplier_id")
VALUES (1, 1), (1, 2);
```

### Crear cliente

**Sentencia**
```python
cliente = Customer.objects.create(
    dni='0923456781',
    first_name='Brithany',
    last_name='Gines',
    email='brithany@example.com',
    phone='0987654321'
)
```

**Resultado**
```text
INSERT INTO "billing_customer" ("dni", "first_name", "last_name", "email", "phone", "address", "photo", "is_active", "created_at", "updated_at")
VALUES ('0923456781', 'Brithany', 'Gines', 'brithany@example.com', '0987654321', NULL, '', 1, '2026-06-24 ...', '2026-06-24 ...')
RETURNING "billing_customer"."id";
```

### Crear perfil OneToOne del cliente

**Sentencia**
```python
perfil = CustomerProfile.objects.create(
    customer=cliente,
    taxpayer_type='ruc',
    payment_terms='credit_30',
    credit_limit=Decimal('800.00')
)
```

**Resultado**
```text
INSERT INTO "billing_customerprofile" ("customer_id", "taxpayer_type", "payment_terms", "credit_limit", "notes")
VALUES (1, 'ruc', 'credit_30', '800.00', NULL)
RETURNING "billing_customerprofile"."id";
```

### Crear factura en borrador

**Sentencia**
```python
factura = Invoice.objects.create(customer=cliente, estado=Invoice.BORRADOR)
```

**Resultado**
```text
INSERT INTO "billing_invoice" ("customer_id", "invoice_date", "subtotal", "tax", "total", "estado", "is_active")
VALUES (1, '2026-06-24 ...', '0.00', '0.00', '0.00', 0, 1)
RETURNING "billing_invoice"."id";
```

### Crear detalle de factura

**Sentencia**
```python
detalle = InvoiceDetail.objects.create(
    invoice=factura,
    product=note,
    quantity=2,
    unit_price=note.unit_price,
    discount_pct=Decimal('5.00')
)
```

**Resultado**
```text
INSERT INTO "billing_invoicedetail" ("invoice_id", "product_id", "quantity", "unit_price", "discount_pct", "subtotal", "tax_amount")
VALUES (1, 1, 2, '289.99', '5.00', '550.98', '82.65')
RETURNING "billing_invoicedetail"."id";
```

### Recalcular totales de la factura

**Sentencia**
```python
factura.subtotal = detalle.subtotal
factura.tax = detalle.tax_amount
factura.total = factura.subtotal + factura.tax
factura.save()
```

**Resultado**
```text
UPDATE "billing_invoice"
SET "customer_id" = 1, "subtotal" = '550.98', "tax" = '82.65', "total" = '633.63', "estado" = 0, "is_active" = 1
WHERE "billing_invoice"."id" = 1;
```

### Crear compra en borrador

**Sentencia**
```python
compra = Purchase.objects.create(supplier=aurora, document_number='AUR-001')
```

**Resultado**
```text
INSERT INTO "purchasing_purchase" ("supplier_id", "document_number", "purchase_date", "subtotal", "tax", "total", "estado", "is_active")
VALUES (1, 'AUR-001', '2026-06-24', '0.00', '0.00', '0.00', 0, 1)
RETURNING "purchasing_purchase"."id";
```

### Crear detalle de compra

**Sentencia**
```python
compra_detalle = PurchaseDetail.objects.create(
    purchase=compra,
    product=note,
    quantity=10,
    unit_cost=Decimal('210.00')
)
```

**Resultado**
```text
INSERT INTO "purchasing_purchasedetail" ("purchase_id", "product_id", "quantity", "unit_cost", "subtotal", "tax_amount")
VALUES (1, 1, 10, '210.00', '2100.00', '315.00')
RETURNING "purchasing_purchasedetail"."id";
```

---

## 10.2 READ

### Consultar todas las marcas

**Sentencia**
```python
Brand.objects.all()
```

**Resultado**
```text
SELECT "billing_brand"."id", "billing_brand"."name", "billing_brand"."description", "billing_brand"."is_active", "billing_brand"."created_at", "billing_brand"."updated_at"
FROM "billing_brand"
ORDER BY "billing_brand"."name" ASC;

<QuerySet [<Brand: Infinix>, <Brand: Xiaomi>]>
```

### Consultar una marca por nombre

**Sentencia**
```python
Brand.objects.get(name='Infinix')
```

**Resultado**
```text
SELECT ... FROM "billing_brand" WHERE "billing_brand"."name" = 'Infinix' LIMIT 21;
<Brand: Infinix>
```

### Filtrar productos mayores a 250

**Sentencia**
```python
Product.objects.filter(unit_price__gt=250)
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE "billing_product"."unit_price" > '250';
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Filtrar productos en rango de precio

**Sentencia**
```python
Product.objects.filter(unit_price__range=(200, 400))
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE "billing_product"."unit_price" BETWEEN '200' AND '400';
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Buscar productos por texto

**Sentencia**
```python
Product.objects.filter(name__icontains='note')
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE "billing_product"."name" LIKE '%note%' ESCAPE '\';
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Excluir productos sin stock

**Sentencia**
```python
Product.objects.exclude(stock=0)
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE NOT ("billing_product"."stock" = 0);
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Ordenar productos por precio

**Sentencia**
```python
Product.objects.order_by('-unit_price')
```

**Resultado**
```text
SELECT ... FROM "billing_product" ORDER BY "billing_product"."unit_price" DESC;
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Contar productos

**Sentencia**
```python
Product.objects.count()
```

**Resultado**
```text
SELECT COUNT(*) AS "__count" FROM "billing_product";
1
```

### Verificar si existen productos sin stock

**Sentencia**
```python
Product.objects.filter(stock=0).exists()
```

**Resultado**
```text
SELECT 1 AS "a" FROM "billing_product" WHERE "billing_product"."stock" = 0 LIMIT 1;
False
```

---

## 10.3 UPDATE

### Editar una marca

**Sentencia**
```python
infinix = Brand.objects.get(name='Infinix')
infinix.description = 'Smartphones y accesorios'
infinix.save()
```

**Resultado**
```text
UPDATE "billing_brand"
SET "name" = 'Infinix', "description" = 'Smartphones y accesorios', "is_active" = 1, "updated_at" = '2026-06-24 ...'
WHERE "billing_brand"."id" = 1;
```

### Actualización masiva

**Sentencia**
```python
Product.objects.filter(stock=0).update(is_active=False)
```

**Resultado**
```text
UPDATE "billing_product" SET "is_active" = 0 WHERE "billing_product"."stock" = 0;
0
```

### Agregar relación ManyToMany

**Sentencia**
```python
note.suppliers.add(elite)
```

**Resultado**
```text
INSERT OR IGNORE INTO "billing_product_suppliers" ("product_id", "supplier_id")
VALUES (1, 2);
```

### Quitar relación ManyToMany

**Sentencia**
```python
note.suppliers.remove(elite)
```

**Resultado**
```text
DELETE FROM "billing_product_suppliers"
WHERE ("billing_product_suppliers"."product_id" = 1 AND "billing_product_suppliers"."supplier_id" IN (2));
```

### Limpiar relaciones ManyToMany

**Sentencia**
```python
note.suppliers.clear()
```

**Resultado**
```text
DELETE FROM "billing_product_suppliers"
WHERE "billing_product_suppliers"."product_id" = 1;
```

### Reemplazar relaciones ManyToMany

**Sentencia**
```python
note.suppliers.set([aurora])
```

**Resultado**
```text
SELECT "billing_supplier"."id" FROM "billing_supplier"
INNER JOIN "billing_product_suppliers" ON ("billing_supplier"."id" = "billing_product_suppliers"."supplier_id")
WHERE "billing_product_suppliers"."product_id" = 1;

INSERT OR IGNORE INTO "billing_product_suppliers" ("product_id", "supplier_id")
VALUES (1, 1);
```

### Editar relación OneToOne

**Sentencia**
```python
cliente = Customer.objects.get(dni='0923456781')
cliente.profile.credit_limit = Decimal('1200.00')
cliente.profile.save()
```

**Resultado**
```text
UPDATE "billing_customerprofile"
SET "customer_id" = 1, "taxpayer_type" = 'ruc', "payment_terms" = 'credit_30', "credit_limit" = '1200.00', "notes" = NULL
WHERE "billing_customerprofile"."id" = 1;
```

### Actualizar precios con F()

**Sentencia**
```python
Product.objects.filter(brand=infinix).update(unit_price=F('unit_price') * Decimal('1.10'))
```

**Resultado**
```text
UPDATE "billing_product"
SET "unit_price" = ("billing_product"."unit_price" * '1.10')
WHERE "billing_product"."brand_id" = 1;
1
```

---

## 10.4 DELETE

### Crear y eliminar una marca sin productos

**Sentencia**
```python
temporal = Brand.objects.create(name='Marca Temporal', description='solo para prueba de delete')
temporal.delete()
```

**Resultado**
```text
INSERT INTO "billing_brand" (...) VALUES ('Marca Temporal', 'solo para prueba de delete', 1, ...);
DELETE FROM "billing_brand" WHERE "billing_brand"."id" = 3;
(1, {'billing.Brand': 1})
```

### Eliminar productos inactivos

**Sentencia**
```python
Product.objects.filter(is_active=False).delete()
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE "billing_product"."is_active" = 0;
DELETE FROM "billing_product" WHERE "billing_product"."is_active" = 0;
(0, {})
```

### Quitar proveedor sin borrar producto ni proveedor

**Sentencia**
```python
note.suppliers.remove(aurora)
```

**Resultado**
```text
DELETE FROM "billing_product_suppliers"
WHERE ("billing_product_suppliers"."product_id" = 1 AND "billing_product_suppliers"."supplier_id" IN (1));
```

---

## 10.5 Relaciones

### ForeignKey desde producto hacia marca

**Sentencia**
```python
note.brand.name
```

**Resultado**
```text
'Infinix'
```

### ForeignKey inversa desde marca hacia productos

**Sentencia**
```python
infinix.products.all()
```

**Resultado**
```text
SELECT ... FROM "billing_product" WHERE "billing_product"."brand_id" = 1;
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### ManyToMany desde producto hacia proveedores

**Sentencia**
```python
note.suppliers.set([aurora, elite])
note.suppliers.all()
```

**Resultado**
```text
SELECT ... FROM "billing_supplier"
INNER JOIN "billing_product_suppliers" ON ("billing_supplier"."id" = "billing_product_suppliers"."supplier_id")
WHERE "billing_product_suppliers"."product_id" = 1;
<QuerySet [<Supplier: Aurora Distribuciones>, <Supplier: Elite Mobile Supply>]>
```

### ManyToMany inversa desde proveedor hacia productos

**Sentencia**
```python
aurora.products.all()
```

**Resultado**
```text
SELECT ... FROM "billing_product"
INNER JOIN "billing_product_suppliers" ON ("billing_product"."id" = "billing_product_suppliers"."product_id")
WHERE "billing_product_suppliers"."supplier_id" = 1;
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### Filtro usando relación ManyToMany

**Sentencia**
```python
Product.objects.filter(suppliers__name='Aurora Distribuciones')
```

**Resultado**
```text
SELECT ... FROM "billing_product"
INNER JOIN "billing_product_suppliers" ON ("billing_product"."id" = "billing_product_suppliers"."product_id")
INNER JOIN "billing_supplier" ON ("billing_product_suppliers"."supplier_id" = "billing_supplier"."id")
WHERE "billing_supplier"."name" = 'Aurora Distribuciones';
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

### OneToOne desde cliente hacia perfil

**Sentencia**
```python
cliente.profile.credit_limit
```

**Resultado**
```text
Decimal('1200.00')
```

### OneToOne desde perfil hacia cliente

**Sentencia**
```python
perfil.customer.full_name
```

**Resultado**
```text
'Brithany Gines'
```

### Filtrar clientes por perfil

**Sentencia**
```python
Customer.objects.filter(profile__taxpayer_type='ruc')
```

**Resultado**
```text
SELECT ... FROM "billing_customer"
INNER JOIN "billing_customerprofile" ON ("billing_customer"."id" = "billing_customerprofile"."customer_id")
WHERE "billing_customerprofile"."taxpayer_type" = 'ruc';
<QuerySet [<Customer: Gines, Brithany>]>
```

### Consulta con Q()

**Sentencia**
```python
Product.objects.filter(Q(brand__name='Infinix') | Q(unit_price__gt=1000))
```

**Resultado**
```text
SELECT ... FROM "billing_product"
INNER JOIN "billing_brand" ON ("billing_product"."brand_id" = "billing_brand"."id")
WHERE ("billing_brand"."name" = 'Infinix' OR "billing_product"."unit_price" > '1000');
<QuerySet [<Product: Infinix Note 40 (Infinix)>]>
```

---

## 10.6 Agregaciones

### Promedio de precios

**Sentencia**
```python
Product.objects.aggregate(avg=Avg('unit_price'))
```

**Resultado**
```text
SELECT AVG("billing_product"."unit_price") AS "avg" FROM "billing_product";
{'avg': Decimal('318.989000000000')}
```

### Precio máximo y mínimo

**Sentencia**
```python
Product.objects.aggregate(max=Max('unit_price'), min=Min('unit_price'))
```

**Resultado**
```text
SELECT MAX("billing_product"."unit_price") AS "max", MIN("billing_product"."unit_price") AS "min" FROM "billing_product";
{'max': Decimal('318.99'), 'min': Decimal('318.99')}
```

### Total facturado por cliente

**Sentencia**
```python
Invoice.objects.filter(customer__dni='0923456781').aggregate(total=Sum('total'))
```

**Resultado**
```text
SELECT SUM("billing_invoice"."total") AS "total"
FROM "billing_invoice"
INNER JOIN "billing_customer" ON ("billing_invoice"."customer_id" = "billing_customer"."id")
WHERE "billing_customer"."dni" = '0923456781';
{'total': Decimal('633.63')}
```

### Contar productos por marca

**Sentencia**
```python
Brand.objects.annotate(n=Count('products')).values('name', 'n')
```

**Resultado**
```text
SELECT "billing_brand"."name" AS "name", COUNT("billing_product"."id") AS "n"
FROM "billing_brand"
LEFT OUTER JOIN "billing_product" ON ("billing_brand"."id" = "billing_product"."brand_id")
GROUP BY "billing_brand"."id", 1;
<QuerySet [{'name': 'Infinix', 'n': 1}, {'name': 'Xiaomi', 'n': 0}]>
```

### Contar proveedores por producto

**Sentencia**
```python
Product.objects.annotate(ns=Count('suppliers')).values('name', 'ns')
```

**Resultado**
```text
SELECT "billing_product"."name" AS "name", COUNT("billing_product_suppliers"."supplier_id") AS "ns"
FROM "billing_product"
LEFT OUTER JOIN "billing_product_suppliers" ON ("billing_product"."id" = "billing_product_suppliers"."product_id")
GROUP BY "billing_product"."id", 1;
<QuerySet [{'name': 'Infinix Note 40', 'ns': 2}]>
```

---

## 10.7 Explicación del flujo en consola y en la página

### Si piden crear una factura

En la página se entra a `Facturas`, se crea una nueva, se selecciona el cliente y se agregan los productos. La factura primero queda en borrador. Cuando ya está revisada, se presiona `Emitir`; ahí el programa cambia el estado a emitida y descuenta el stock.

En consola se hace lo mismo con ORM:

**Sentencia**
```python
factura = Invoice.objects.create(customer=cliente, estado=Invoice.BORRADOR)
detalle = InvoiceDetail.objects.create(
    invoice=factura,
    product=note,
    quantity=2,
    unit_price=note.unit_price,
    discount_pct=Decimal('5.00')
)

factura.subtotal = detalle.subtotal
factura.tax = detalle.tax_amount
factura.total = factura.subtotal + factura.tax
factura.save()

Product.objects.filter(pk=note.pk).update(stock=F('stock') - detalle.quantity)
factura.estado = Invoice.EMITIDA
factura.save()
```

**Resultado**
```text
Se crea la cabecera de la factura, se guarda el detalle, se calculan subtotal, IVA y total.
Después se descuenta el stock del producto y la factura pasa de Borrador a Emitida.
```

### Si piden anular una factura

Una factura emitida no se debe borrar, porque ya forma parte del historial. En la página se usa `Anular`; en consola se devuelve el stock y se cambia el estado.

**Sentencia**
```python
Product.objects.filter(pk=note.pk).update(stock=F('stock') + detalle.quantity)
factura.estado = Invoice.ANULADA
factura.is_active = False
factura.save()
```

**Resultado**
```text
El stock vuelve al producto y la factura queda Anulada.
```

### Si piden agregar una nueva compra

En la página se entra a `Compras`, se crea una nueva compra, se escoge el proveedor, el número de documento y los productos comprados. Al inicio queda como borrador. Cuando se confirma, el programa aumenta el stock.

En consola se representa así:

**Sentencia**
```python
compra = Purchase.objects.create(supplier=aurora, document_number='AUR-001')
compra_detalle = PurchaseDetail.objects.create(
    purchase=compra,
    product=note,
    quantity=10,
    unit_cost=Decimal('210.00')
)

compra.subtotal = compra_detalle.subtotal
compra.tax = compra_detalle.tax_amount
compra.total = compra.subtotal + compra.tax
Product.objects.filter(pk=note.pk).update(stock=F('stock') + compra_detalle.quantity)
compra.estado = Purchase.CONFIRMADA
compra.save()
```

**Resultado**
```text
Se registra la compra, se calculan los totales y el producto aumenta su stock.
La compra pasa de Borrador a Confirmada.
```

### Resumen para explicar

La factura representa una venta, por eso al emitirla baja el inventario. La compra representa entrada de mercadería, por eso al confirmarla sube el inventario. En ambos casos el ORM usa los modelos del proyecto y guarda los cambios en la misma base de datos que usa la página.

---

## Práctica adicional para la lección

### Editar producto desde consola

**Sentencia**
```python
note.stock = 40
note.unit_price = Decimal('299.99')
note.save()
```

**Resultado**
```text
UPDATE "billing_product"
SET "stock" = 40, "unit_price" = '299.99', ...
WHERE "billing_product"."id" = 1;
```

### Emitir factura correctamente desde ORM

**Sentencia**
```python
Product.objects.filter(pk=note.pk).update(stock=F('stock') - detalle.quantity)
factura.estado = Invoice.EMITIDA
factura.save()
```

**Resultado**
```text
UPDATE "billing_product" SET "stock" = ("billing_product"."stock" - 2) WHERE "billing_product"."id" = 1;
UPDATE "billing_invoice" SET "estado" = 1, ... WHERE "billing_invoice"."id" = 1;
```

### Anular factura correctamente desde ORM

**Sentencia**
```python
Product.objects.filter(pk=note.pk).update(stock=F('stock') + detalle.quantity)
factura.estado = Invoice.ANULADA
factura.is_active = False
factura.save()
```

**Resultado**
```text
UPDATE "billing_product" SET "stock" = ("billing_product"."stock" + 2) WHERE "billing_product"."id" = 1;
UPDATE "billing_invoice" SET "estado" = 2, "is_active" = 0, ... WHERE "billing_invoice"."id" = 1;
```

### Crear nota de crédito de factura

**Sentencia**
```python
nota = CreditNote.objects.create(
    invoice=factura,
    tipo=CreditNote.TIPO_TOTAL,
    amount=factura.total,
    reason='Anulación total de la venta'
)
```

**Resultado**
```text
INSERT INTO "billing_creditnote" ("invoice_id", "date", "tipo", "amount", "reason", "is_active")
VALUES (1, '2026-06-24', 'total', '633.63', 'Anulación total de la venta', 1)
RETURNING "billing_creditnote"."id";
```

### Confirmar compra correctamente desde ORM

**Sentencia**
```python
Product.objects.filter(pk=note.pk).update(stock=F('stock') + compra_detalle.quantity)
compra.estado = Purchase.CONFIRMADA
compra.subtotal = compra_detalle.subtotal
compra.tax = compra_detalle.tax_amount
compra.total = compra.subtotal + compra.tax
compra.save()
```

**Resultado**
```text
UPDATE "billing_product" SET "stock" = ("billing_product"."stock" + 10) WHERE "billing_product"."id" = 1;
UPDATE "purchasing_purchase" SET "subtotal" = '2100.00', "tax" = '315.00', "total" = '2415.00', "estado" = 1, ...
WHERE "purchasing_purchase"."id" = 1;
```

### Eliminar solo borradores

**Sentencia**
```python
Invoice.objects.filter(estado=Invoice.BORRADOR).delete()
```

**Resultado**
```text
DELETE FROM "billing_invoice" WHERE "billing_invoice"."estado" = 0;
(0, {})
```

> En la vida real no se debe eliminar una factura emitida. Si ya fue emitida, se anula y queda como historial.
