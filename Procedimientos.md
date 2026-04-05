CREATE PROCEDURE sp_ObtenerEstudiosSinBeca
AS
BEGIN
    SET NOCOUNT ON;

    SELECT 
        e.id, 
        e.titulo
    FROM estudios_realizados e
    LEFT JOIN beca b ON e.id = b.estudios
    WHERE b.estudios IS NULL;
END
GO

-- Insertar 
CREATE PROCEDURE sp_insertar_intereses_futuros
    @docente INT,
    @termino_clave NVARCHAR(30)
AS
BEGIN
    INSERT INTO intereses_futuros (docente, termino_clave)
    VALUES (@docente, @termino_clave);
END;

-- Eliminar (toma ambas claves)
CREATE PROCEDURE sp_eliminar_intereses_futuros
    @docente INT,
    @termino_clave NVARCHAR(30)
AS
BEGIN
    DELETE FROM intereses_futuros
    WHERE docente = @docente AND termino_clave = @termino_clave;
END;

-- Para "editar": borrar y crear nuevo 
CREATE PROCEDURE sp_actualizar_intereses_futuros
    @docente_viejo INT,
    @termino_clave_viejo NVARCHAR(30),
    @docente_nuevo INT,
    @termino_clave_nuevo NVARCHAR(30)
AS
BEGIN
    -- Eliminar el viejo
    DELETE FROM intereses_futuros
    WHERE docente = @docente_viejo AND termino_clave = @termino_clave_viejo;
    
    -- Insertar el nuevo
    INSERT INTO intereses_futuros (docente, termino_clave)
    VALUES (@docente_nuevo, @termino_clave_nuevo);
END;