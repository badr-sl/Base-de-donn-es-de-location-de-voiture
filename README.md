# Base de donnes de location de voiture
Cette base de données a été conçue pour gérer le processus de location de voitures. Elle utilise le système de gestion de base de données SQL Server et est écrite en T-SQL.
#Installation
CREATE DATABASE  location;
GO
USE location;
GO
CREATE TABLE Clients (
Id INT PRIMARY KEY,
Nom VARCHAR(50) NOT NULL,
Adresse VARCHAR(100),
Ville VARCHAR(50),
Pays VARCHAR(50)
);

-- Create the Voitures table
CREATE TABLE Voitures (
Id INT PRIMARY KEY,
Marque VARCHAR(50) NOT NULL,
Modele VARCHAR(50) NOT NULL,
Annee INT NOT NULL,
Couleur VARCHAR(50),
Disponible BIT DEFAULT 1,
PrixJournalier DECIMAL(10, 2),
NombrePlaces INT,
Carburant VARCHAR(50),
Transmission VARCHAR(50),
GPS BIT DEFAULT 0
);

CREATE TABLE Locations (
Id INT PRIMARY KEY,
IdClient INT FOREIGN KEY REFERENCES Clients(Id),
IdVoiture INT FOREIGN KEY REFERENCES Voitures(Id),
DateDebut DATE NOT NULL,
DateFin DATE NOT NULL,
Montant DECIMAL(10, 2)
);

-- Create the Agences table
CREATE TABLE Agences (
Id INT PRIMARY KEY,
Nom VARCHAR(50) NOT NULL,
Adresse VARCHAR(100),
Ville VARCHAR(50),
Pays VARCHAR(50),
Telephone VARCHAR(20),
Email VARCHAR(50)
);
-- Create the Factures table
CREATE TABLE Factures (
    Id INT PRIMARY KEY,
    IdLocation INT FOREIGN KEY REFERENCES Locations(Id),
    DateFacture DATE NOT NULL,
    MontantTotal DECIMAL(10, 2) NOT NULL);

CREATE TABLE ReservationsEnLigne (
Id INT PRIMARY KEY,
IdClient INT FOREIGN KEY REFERENCES Clients(Id),
DateReservation DATETIME NOT NULL,
DateDebut DATE NOT NULL,
DateFin DATE NOT NULL,
IdVoiture INT FOREIGN KEY REFERENCES Voitures(Id),
Statut VARCHAR(50),
PrixTotal DECIMAL(10, 2)
);

CREATE TABLE Employes (
Id INT PRIMARY KEY,
Nom VARCHAR(50) NOT NULL,
Prenom VARCHAR(50) NOT NULL,
Adresse VARCHAR(100),
Ville VARCHAR(50),
Pays VARCHAR(50),
DateEmbauche DATE NOT NULL,
Salaire DECIMAL(10, 2) NOT NULL,
Fonction VARCHAR(50),
Email VARCHAR(50),
Telephone VARCHAR(20),
IdAgence INT,
CONSTRAINT fk_agence FOREIGN KEY (IdAgence) REFERENCES Agences(Id)
);




-- Insert sample data into the Clients table
INSERT INTO Clients (Id, Nom, Adresse, Ville, Pays)
VALUES (1, 'Dupont', '10 Rue de la Paix', 'Paris', 'France'),
(2, 'Smith', '123 Main St', 'New York', 'USA'),
(3, 'Kim', '456 Maple Ave', 'Seoul', 'South Korea');

-- Insert sample data into the Voitures table
INSERT INTO Voitures (Id, Marque, Modele, Annee, Couleur, Disponible, PrixJournalier, NombrePlaces, Carburant, Transmission, GPS)
VALUES (1, 'Toyota', 'Corolla', 2021, 'Noir', 1, 50.00, 5, 'Essence', 'Automatique', 1),
(2, 'Honda', 'Civic', 2020, 'Blanc', 1, 60.00, 5, 'Essence', 'Manuelle', 0),
(3, 'Nissan', 'Altima', 2019, 'Rouge', 1, 70.00, 5, 'Essence', 'Automatique', 1);

-- Insert sample data into the Locations table
INSERT INTO Locations (Id, IdClient, IdVoiture, DateDebut, DateFin, Montant)
VALUES (1, 1, 1, '2023-05-01', '2023-05-05', 200.00),
(2, 2, 2, '2023-06-15', '2023-06-22', 420.00),
(3, 3, 3, '2023-07-10', '2023-07-15', 350.00);

INSERT INTO Agences (Id, Nom, Adresse, Ville, Pays, Telephone, Email)
VALUES (1, 'Agence A', '10 Rue de la Paix', 'Paris', 'France', '+33 1 23 45 67 89', 'agencea@example.com'),
(2, 'Agence B', '123 Main St', 'New York', 'USA', '+1 212-555-1212', 'agenceb@example.com'),
(3, 'Agence C', '456 Maple Ave', 'Toronto', 'Canada', '+1 416-555-1212', 'agencec@example.com'),
 (4, 'Agence D', '1 Chome-1-1 Nishi-Shinjuku', 'Tokyo', 'Japan', '+81 3-1234-5678', 'agenced@example.com');

 INSERT INTO Factures (Id, IdLocation, DateFacture, MontantTotal)
VALUES (1, 1, '2023-05-05', 200.00),
       (2, 2, '2023-06-22', 420.00),
       (3, 3, '2023-07-15', 350.00);
INSERT INTO Employes (Id, Nom, Prenom, Adresse, Ville, Pays, DateEmbauche, Salaire, Fonction, Email, Telephone, IdAgence)
VALUES
(1, 'Doe', 'John', '123 Main St', 'New York', 'USA', '2020-01-01', 5000, 'Manager', 'johndoe@email.com', '123-456-7890', 1),
(2, 'Smith', 'Jane', '456 Second Ave', 'San Francisco', 'USA', '2018-05-01', 4000, 'Assistant Manager', 'janesmith@email.com', '555-555-5555', 2),
(3, 'Johnson', 'David', '789 Third St', 'Chicago', 'USA', '2019-03-15', 4500, 'Sales Representative', 'davidjohnson@email.com', '777-777-7777', 3);

INSERT INTO ReservationsEnLigne (Id, IdClient, DateReservation, DateDebut, DateFin, IdVoiture, Statut, PrixTotal) 
VALUES (1, 1, '2023-05-10 10:00:00', '2023-05-11', '2023-05-15', 1, 'CONFIRMED', 500.00),
       (2, 2, '2023-05-12 09:30:00', '2023-05-20', '2023-05-25', 3, 'PENDING', 750.00),
       (3, 3, '2023-05-13 14:15:00', '2023-05-19', '2023-05-23', 2, 'CANCELLED', 0.00),
       (4, 4, '2023-05-15 16:00:00', '2023-05-25', '2023-05-28', 5, 'PENDING', 450.00),
       (5, 5, '2023-05-17 11:45:00', '2023-05-22', '2023-05-24', 6, 'CONFIRMED', 300.00);
ALTER TABLE Voitures
ADD NombreReservations INT DEFAULT 0;
GO






-- Créer le déclencheur
CREATE TRIGGER UpdateReservationCount
ON Locations
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    -- Mettre à jour le nombre de réservations pour chaque voiture
    UPDATE Voitures
    SET NombreReservations = (
        SELECT COUNT(*) FROM Locations WHERE Locations.IdVoiture = Voitures.Id
    )
    FROM Voitures
    INNER JOIN (
        SELECT IdVoiture FROM inserted
        UNION
        SELECT IdVoiture FROM deleted
    ) AS Changes ON Voitures.Id = Changes.IdVoiture;
END;
GO

-- Declare variables
DECLARE @ClientId INT,
        @ClientNom VARCHAR(50),
        @ClientAdresse VARCHAR(100),
        @ClientVille VARCHAR(50),
        @ClientPays VARCHAR(50);

-- Declare the cursor
DECLARE client_cursor CURSOR FOR
SELECT Id, Nom, Adresse, Ville, Pays
FROM Clients;

-- Open the cursor
OPEN client_cursor;

-- Fetch the first row
FETCH NEXT FROM client_cursor INTO @ClientId, @ClientNom, @ClientAdresse, @ClientVille, @ClientPays;

-- Start the loop
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Perform operations using the fetched data
    
    -- Print the client information
    PRINT 'Client ID: ' + CONVERT(VARCHAR(10), @ClientId);
    PRINT 'Client Name: ' + @ClientNom;
    PRINT 'Client Address: ' + ISNULL(@ClientAdresse, '');
    PRINT 'Client City: ' + ISNULL(@ClientVille, '');
    PRINT 'Client Country: ' + ISNULL(@ClientPays, '');
    PRINT '';

    -- Fetch the next row
    FETCH NEXT FROM client_cursor INTO @ClientId, @ClientNom, @ClientAdresse, @ClientVille, @ClientPays;
END

-- Close the cursor
CLOSE client_cursor;

-- Deallocate the cursor
DEALLOCATE client_cursor;



-- Create the stored procedure
CREATE PROCEDURE ProcessClients
AS
BEGIN
    -- Declare variables
    DECLARE @ClientId INT,
            @ClientNom VARCHAR(50),
            @ClientAdresse VARCHAR(100),
            @ClientVille VARCHAR(50),
            @ClientPays VARCHAR(50);

    -- Declare the cursor
    DECLARE client_cursor CURSOR FOR
    SELECT Id, Nom, Adresse, Ville, Pays
    FROM Clients;

    -- Open the cursor
    OPEN client_cursor;

    -- Fetch the first row
    FETCH NEXT FROM client_cursor INTO @ClientId, @ClientNom, @ClientAdresse, @ClientVille, @ClientPays;

    -- Start the loop
    WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Perform operations using the fetched data
        
        -- Print the client information
        PRINT 'Client ID: ' + CONVERT(VARCHAR(10), @ClientId);
        PRINT 'Client Name: ' + @ClientNom;
        PRINT 'Client Address: ' + ISNULL(@ClientAdresse, '');
        PRINT 'Client City: ' + ISNULL(@ClientVille, '');
        PRINT 'Client Country: ' + ISNULL(@ClientPays, '');
        PRINT '';

        -- Fetch the next row
        FETCH NEXT FROM client_cursor INTO @ClientId, @ClientNom, @ClientAdresse, @ClientVille, @ClientPays;
    END

    -- Close the cursor
    CLOSE client_cursor;

    -- Deallocate the cursor
    DEALLOCATE client_cursor;
END
EXEC ProcessClients;
