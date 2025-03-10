*eoAPI**

1. **Setup**
    1. **Install and start Docker Compose**

Have look at <https://docs.docker.com/compose/install/>

**1.2 Clone the Repository**

Start by cloning the eoAPI GitHub repository:  

Commands:  
git clone <https://github.com/developmentseed/eoAPI.git>  
cd eoAPI

- 1. **Configure Environment Variables**
- Create a .env file in the root directory of the project (if not already present).
- Add the required environment variables.
- Update the .env file with appropriate values for:
  - Database credentials (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB).
  - Additional service configurations, if needed.

**1.4 Build and Run Services**

Use docker-compose to build and start all the services:

``` 
docker-compose up --build
``` 

This command will:

- Build the Docker containers for all the services (stac-browser, stac-fastapi, titiler-pgstac, tipg, and the database).
- Start the containers.

**1.5 Access the Services**

- **STAC Browser**: Visit <http://localhost:8085> to browse STAC collections.
- **STAC FastAPI**: The STAC API backend is accessible at <http://localhost:8081>.
- **TiTiler**: Use <http://localhost:8082> for dynamic tiling and rendering.
- **TIPG**: The OGC API Features are available at <http://localhost:8083>.

** 2. Explanation of the Docker Container**

2.1. **stac-browser**:

- **Purpose**: Provides a web-based interface for browsing SpatioTemporal Asset Catalogs (STAC).
- **Details**: This service is built from a custom Dockerfile (Dockerfile.browser) located in the dockerfiles directory. It exposes port 8085 and depends on the stac-fastapi, titiler-pgstac, and database services to function correctly.

2.2. **stac-fastapi**:

- **Purpose**: Serves as the API backend for STAC, allowing users to search and interact with STAC items and collections.
- **Details**: This service uses the image ghcr.io/stac-utils/stac-fastapi-pgstac:3.0.0 and exposes port 8081. It connects to the PostgreSQL database (database service) using environment variables that define the database connection parameters.

2.3. **titiler-pgstac**:

- **Purpose**: Provides dynamic tiling and rendering of geospatial data stored in the STAC database.
- **Details**: This service uses the image ghcr.io/developmentseed/titiler-pgstac:latest and exposes port 8082. It also connects to the PostgreSQL database (database service) to access geospatial data.

2.4. **tipg**:

- **Purpose**: Offers OGC API Features, enabling access to vector data through standardized OGC API endpoints.
- **Details**: This service uses the image ghcr.io/developmentseed/tipg:latest and exposes port 8083. It connects to the PostgreSQL database (database service) to serve vector data.

2.5. **database**:

- **Purpose**: Hosts the PostgreSQL database with PostGIS extensions, serving as the primary data store for the application.
- **Details**: This service uses the image postgis/postgis:15-3.3 and exposes port 5432. It initializes with environment variables that set the database name, user, and password.


**3. Ingest raster data into database**

3.1 **Install pypgstac**

``` 
pip install pypgstac
``` 

3.2. **Insert Collection**

``` 
pypgstac load collections collection.json --dsn postgresql://username:password@0.0.0.0:5439/postgis
``` 

3.3. **Insert Items**

``` 
pypgstac load items item.json --dsn postgresql://username:password@0.0.0.0:5439/postgis
``` 
3.4. **Example**

```
pypgstac load collections data/cop-dem-30/collection.json --dsn postgresql://username:password@0.0.0.0:5439/postgis
pypgstac load items data/cop-dem-30/Copernicus_DSM_COG_10_S01_00_E030_00_DEM/Copernicus_DSM_COG_10_S01_00_E030_00_DEM.json --dsn postgresql://username:password@0.0.0.0:5439/postgis
```

3.5. **Test if it worked**

Visit http://localhost:8085/collections/cop-dem-glo-30


**4. Ingest vector data into database**

4.1. **Convert shapefile to sql**

```
shp2pgsql nationalparks.shp public.nationalparks > nationalparks.sql
```

4.2. **Insert into database**

```
psql -d postgresql://username:password@0.0.0.0:5439/postgis -f nationalparks.sql
```

4.3. **Test if it worked**

Visit http://localhost:8083/collections/public.nationalparks
