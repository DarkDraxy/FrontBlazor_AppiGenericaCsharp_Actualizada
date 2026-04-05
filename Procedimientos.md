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

