# Solución examen simulacro de plpgsql

## 1. Función

    CREATE OR REPLACE FUNCTION verificar_email_unico() RETURNS TRIGGER AS $$
    BEGIN
        IF EXISTS (SELECT 1 FROM Clientes WHERE email = NEW.email) THEN
            RAISE EXCEPTION 'El correo electrónico ya existe.';
        END IF;
        RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;
    
## 1. Trigger

      CREATE TRIGGER trigger_verificar_email_unico 
      BEFORE INSERT ON Clientes 
      FOR EACH ROW EXECUTE PROCEDURE verificar_email_unico();
      
## 2. Función

    CREATE OR REPLACE FUNCTION actualizar_fecha_modificacion() RETURNS TRIGGER AS $$
    BEGIN
        NEW.fecha_ultima_modificacion = NOW();
        RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;
    
## 2. Trigger

      CREATE TRIGGER trigger_actualizar_fecha_modificacion 
      BEFORE UPDATE ON Pedidos 
      FOR EACH ROW EXECUTE PROCEDURE actualizar_fecha_modificacion();
      
## 3. Función

NOTA: La variable TG_OP es una variable de sistema en PL/pgSQL que se proporciona automáticamente en el contexto de un disparador (trigger).

La variable TG_OP se utiliza para determinar qué operación causó la activación del disparador. Puede tener uno de los siguientes valores:

'INSERT'
'UPDATE'
'DELETE'
'TRUNCATE'

    CREATE OR REPLACE FUNCTION registrar_cambios() RETURNS TRIGGER AS $$
    BEGIN
        IF (TG_OP = 'UPDATE') THEN
            IF (OLD.producto != NEW.producto) THEN
                INSERT INTO LogCambios(id_pedido, campo_modificado, valor_antiguo, valor_nuevo) 
                VALUES (OLD.id, 'producto', OLD.producto, NEW.producto);
            END IF;
            IF (OLD.cantidad != NEW.cantidad) THEN
                INSERT INTO LogCambios(id_pedido, campo_modificado, valor_antiguo, valor_nuevo) 
                VALUES (OLD.id, 'cantidad', CAST(OLD.cantidad AS VARCHAR(100)), CAST(NEW.cantidad AS VARCHAR(100)));
            END IF;
        END IF;
        RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;
    
## 3. Trigger

    CREATE TRIGGER trigger_registrar_cambios 
    AFTER UPDATE ON Pedidos 
    FOR EACH ROW EXECUTE PROCEDURE registrar_cambios();
    
    
## 4. Teoría

    DECLARE
        cursor_pedidos CURSOR FOR SELECT * FROM Pedidos;

    OPEN cursor_pedidos;

    FETCH NEXT FROM cursor_pedidos INTO @pedido;
    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Aquí puedes procesar cada fila. En este caso, solo vamos a imprimir el pedido
        RAISE NOTICE '%', @pedido;
        FETCH NEXT FROM cursor_pedidos INTO @pedido;
    END;

    CLOSE cursor_pedidos;
