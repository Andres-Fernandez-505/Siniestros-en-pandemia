CREATE TABLE backup_accidentes AS SELECT * FROM accidentes;

ALTER TABLE accidentes CHANGE nro_acci acci_cod INT;

ALTER TABLE accidentes ADD PRIMARY KEY (acci_cod);

CREATE TABLE EstadoClima(
	clim_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	clim_desc VARCHAR(45)
);

CREATE TABLE tipozona (
	tpzo_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	tpzo_desc VARCHAR(45)
);

CREATE TABLE tipo_calle (
	tpca_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	tpca_desc VARCHAR(45)
);

CREATE TABLE Dias (
	dia_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	dia_desc VARCHAR(45)
);

CREATE TABLE Participantes (
	part_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	part_desc VARCHAR(45)
);

CREATE TABLE Departamentos (
	dept_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	dept_desc VARCHAR(45)
);

CREATE TABLE Localidades (
	loca_cod INT(11) PRIMARY KEY AUTO_INCREMENT,
	loca_desc VARCHAR(45),
	dept_cod INT (11),
	CONSTRAINT fk_dept_cod FOREIGN KEY (dept_cod) REFERENCES departamentos (dept_cod)
);

CREATE TABLE Accidente_has_Participantes (
    acci_cod INT,
    part_cod INT,
    CONSTRAINT pk_acci_part PRIMARY KEY (acci_cod, part_cod),
    CONSTRAINT fk_acci_cod FOREIGN KEY (acci_cod) REFERENCES accidentes(acci_cod) ,
    CONSTRAINT fk_part_cod FOREIGN KEY (part_cod) REFERENCES participantes(part_cod)
);


INSERT INTO dias (dia_desc)
SELECT DISTINCT desc_dia
FROM accidentes;

INSERT INTO tipozona (tpzo_desc)
SELECT DISTINCT desc_zona
FROM accidentes;

INSERT INTO tipo_calle (tpca_desc)
SELECT DISTINCT desc_tipo_via
FROM accidentes;

INSERT INTO estadoclima (clim_desc)
SELECT DISTINCT desc_estado_clima
FROM accidentes;

INSERT INTO departamentos (dept_desc)
SELECT DISTINCT desc_dpto
FROM accidentes;

INSERT INTO Localidades (loca_desc, dept_cod)
SELECT DISTINCT a.desc_loc, d.dept_cod
FROM accidentes a
JOIN Departamentos d ON a.desc_dpto = d.dept_desc; 

INSERT INTO Participantes (part_desc)
SELECT DISTINCT SUBSTRING_INDEX(SUBSTRING_INDEX(a.desc_participante, ',', n.n), ',', -1)
FROM accidentes a
JOIN (SELECT 1 AS n UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6) n
WHERE CHAR_LENGTH(a.desc_participante) - CHAR_LENGTH(REPLACE(a.desc_participante, ',', '')) >= n.n - 1;

INSERT INTO accidente_has_participantes (acci_cod, part_cod)
SELECT DISTINCT a.acci_cod, p.part_cod
FROM accidentes a
JOIN (SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 ) n ON n.n <= 1 + LENGTH(a.desc_participante) - LENGTH(REPLACE(a.desc_participante, ',', ''))
JOIN participantes p ON SUBSTRING_INDEX(SUBSTRING_INDEX(a.desc_participante, ',', n.n), ',', -1) = p.part_desc;

ALTER TABLE accidentes
ADD COLUMN desc_loc_backup VARCHAR(255);

UPDATE accidentes
SET desc_loc_backup = desc_loc;

UPDATE accidentes a
JOIN localidades l ON TRIM(LOWER(a.desc_loc)) = TRIM(LOWER(l.loca_desc))
SET a.desc_loc = l.loca_cod;

ALTER TABLE accidentes
MODIFY COLUMN desc_loc INT(11);

ALTER TABLE accidentes
ADD CONSTRAINT fk_accidentes_loca_cod
FOREIGN KEY (desc_loc) REFERENCES localidades(loca_cod);


ALTER TABLE accidentes
ADD COLUMN desc_dia_backup VARCHAR(50);

UPDATE accidentes
SET desc_dia_backup = desc_dia;

ALTER TABLE accidentes
MODIFY COLUMN desc_dia INT;

UPDATE accidentes a
JOIN dias d ON TRIM(LOWER(a.desc_dia_backup)) = TRIM(LOWER(d.dia_desc))
SET a.desc_dia = d.dia_cod;

ALTER TABLE accidentes
ADD CONSTRAINT fk_accidentes_dia_cod
FOREIGN KEY (desc_dia) REFERENCES dias(dia_cod);


ALTER TABLE accidentes
DROP COLUMN anio_acci,
DROP COLUMN desc_ruta,
DROP COLUMN km,
DROP COLUMN cant_participantes,
DROP COLUMN heridos_gravisimos,
DROP COLUMN sin_datos,
DROP COLUMN posicion_XY,
DROP COLUMN desc_ruta_ori,
DROP COLUMN desc_tipo_calzada,
DROP COLUMN desc_tipo_banquina,
DROP COLUMN desc_unidad_regional,
DROP COLUMN desc_lugar_calzada,
DROP COLUMN desc_prioridad,
DROP COLUMN desc_estado_semaforo,
DROP COLUMN desc_lugar_via,
DROP COLUMN desc_estado_via,
DROP COLUMN desc_estado_visibilidad,
DROP COLUMN desc_luminosidad,
DROP COLUMN desc_tipo_colision,
DROP COLUMN desc_tipo_atropello,
DROP COLUMN desc_tipo_hecho,
DROP COLUMN desc_pres_calzada,
DROP COLUMN desc_senializacion,
DROP COLUMN desc_separacion_via,
DROP COLUMN desc_restriccion;

ALTER TABLE accidentes ADD COLUMN tpzo_cod INT;

UPDATE accidentes a
JOIN tipozona tz ON TRIM(LOWER(a.desc_zona)) = TRIM(LOWER(tz.tpzo_desc))
SET a.tpzo_cod = tz.tpzo_cod;

ALTER TABLE accidentes DROP COLUMN desc_zona;

ALTER TABLE accidentes
ADD CONSTRAINT fk_accidentes_tpzo_cod
FOREIGN KEY (tpzo_cod) REFERENCES tipozona(tpzo_cod);

ALTER TABLE accidentes ADD COLUMN tpca_cod INT;

UPDATE accidentes a
JOIN tipo_calle tc ON TRIM(LOWER(a.desc_tipo_via)) = TRIM(LOWER(tc.tpca_desc))
SET a.tpca_cod = tc.tpca_cod;

ALTER TABLE accidentes DROP COLUMN desc_tipo_via;

ALTER TABLE accidentes
ADD CONSTRAINT fk_accidentes_tpca_cod
FOREIGN KEY (tpca_cod) REFERENCES tipo_calle(tpca_cod);

ALTER TABLE accidentes ADD COLUMN clim_cod INT;

UPDATE accidentes a
JOIN estadoclima ec ON TRIM(LOWER(a.desc_estado_clima)) = TRIM(LOWER(ec.clim_desc))
SET a.clim_cod = ec.clim_cod;

ALTER TABLE accidentes DROP COLUMN desc_estado_clima;

ALTER TABLE accidentes
ADD CONSTRAINT fk_accidentes_clim_cod
FOREIGN KEY (clim_cod) REFERENCES estadoclima(clim_cod);

UPDATE accidentes
SET hora_aprox = NULL
WHERE hora_aprox = '';

UPDATE accidentes
SET calle_avenida_km = 'S/D'
WHERE calle_avenida_km = '';

ALTER TABLE accidentes CHANGE fecha acci_fech DATE;
ALTER TABLE accidentes CHANGE hora_aprox acci_hora TIME;
ALTER TABLE accidentes CHANGE desc_dia dia_cod INT;
   
   
ALTER TABLE accidentes CHANGE total acci_total INT;
   
ALTER TABLE accidentes CHANGE heridos_leves acci_leves INT;
   
ALTER TABLE accidentes CHANGE heridos_graves acci_graves INT;
   
ALTER TABLE accidentes CHANGE ilesos acci_ilesos INT;
   
ALTER TABLE accidentes CHANGE fallecidos acci_falle INT;
   
ALTER TABLE accidentes CHANGE desc_participante acci_part VARCHAR(200);

ALTER TABLE accidentes CHANGE calle_avenida_km acci_ubic VARCHAR(200);

ALTER TABLE accidentes
DROP COLUMN desc_dpto,
DROP COLUMN desc_loc_backup,
DROP COLUMN desc_dia_backup;


