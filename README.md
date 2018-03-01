/****** Object:  StoredProcedure [dbo].[sp_POPropertySearch]    Script Date: 01-Mar-18 1:29:28 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
-- =============================================
-- Author:      <Roopa Anumula>
-- Create Date: <22-Feb-2018 >
-- Description: <Develop Stored Procedure for PO Property with search, sort and pagination>
-- =============================================
ALTER PROCEDURE [dbo].[sp_POPropertySearch]
(
        -- Add the parameters for the stored procedure here
    @SearchString nvarchar(max),
	@PageNum INT,
	@SortString nvarchar(20),
	@Ownerid INT
	  
)
AS
BEGIN

 --Sort by Name in Ascending Order
 
		Select P.Name, 
	    Concat(A.Number,' ',A.Street,' ', A.Suburb,' ',A.city,' ',A.PostCode) As FullAddress,
		PT.Name As PropertyType,
		Concat(p.Bedroom,' Bedrooms ',P.Bathroom,' Bathrooms ',P.ParkingSpace,' Parking ') As Rooms,
		p.LandSqm As LandArea,
		P.FloorArea,
		P.TargetRent, 
		P.YearBuilt,
		case when P.IsOwnerOccupied = 1 
		then 'Yes'
		else 
		 'No' end as OwnerCccupied,
		
		P.Description,
		A.Lat,
		A.Lng,
		PM.Id as MediaId,
		PM.NewFileName as NewFileName,
		PM.OldFileName as OldFileName
		from OwnerProperty OP
		inner join Property P on OP.PropertyId = P.Id
		inner join Address A on P.AddressId = A.AddressId
		full outer join PropertyType PT on P.PropertyTypeId = PT.PropertyTypeId
		Full outer join PropertyMedia PM on OP.PropertyId = PM .PropertyId 
		WHERE  
			   (P.Name LIKE '%'+@SearchString + '%')
			   OR Concat(A.Number,' ',A.Street,' ', A.Suburb,' ',A.city,' ',A.PostCode) like '%SearchString%'
			   OR OP.OwnerId = @Ownerid
			   and P.IsActive=1
			   and A.IsActive=1

		Order by 
		Case @SortString when 'Name' Then P.Name End Asc
		,Case @SortString when 'Name(Desc)' Then P.Name End Desc
		,Case @SortString when 'Latest Date' Then OP.PurchaseDate End Asc
		,Case @SortString when 'Earliest Date' Then OP.PurchaseDate End Desc
		
		OFFSET (@PageNum - 1) * 10  ROWS
		FETCH NEXT 10  ROWS ONLY
END


