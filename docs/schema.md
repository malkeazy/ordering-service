# CREATE DATABASE orderingservice;

## products
```sql
CREATE TABLE products
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
    name VARCHAR(100) NOT NULL
    in_stock INTEGER NOT NULL CHECK (in_stock >= 0)
    price NUMERIC(15, 2) NOT NULL CHECK (price >= 0)
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
```
## orders
```sql
CREATE TABLE orders
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
    status VARCHAR(20) NOT NULL DEFAULT 'created' CHECK (status in ('created', 'paid', 'cancelled'))
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()   
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
    total_amount NUMERIC(15,2) NOT NULL CHECK (total_amount >= 0)
```
## order_items
```sql
CREATE TABLE order_items
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id BIGINT NOT NULL REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(15, 2) NOT NULL CHECK (unit_price >= 0)
    UNIQUE (order_id, product_id)
    CREATE INDEX ON order_items (product_id);
```

## trigers
```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER orders_set_updated_at
    BEFORE UPDATE ON orders
    FOR EACH ROW
    EXECUTE FUNCTION set_updated_at();
```
