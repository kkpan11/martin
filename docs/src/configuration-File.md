# Configuration File

If you don't want to expose all of your tables and functions, you can list your sources in a configuration file. To start Martin with a configuration file you need to pass a path to a file with a `--config` argument. Config files may contain environment variables, which will be expanded before parsing. For example, to use `MY_DATABASE_URL` in your config file: `connection_string: ${MY_DATABASE_URL}`, or with a default `connection_string: ${MY_DATABASE_URL:-postgresql://postgres@localhost/db}`

```shell
martin --config config.yaml
```

You may wish to auto-generate a config file with `--save-config` argument. This will generate a config yaml file with all of your configuration, which you can edit to remove any sources you don't want to expose.

```yaml
# Connection keep alive timeout [default: 75]
keep_alive: 75

# The socket address to bind [default: 0.0.0.0:3000]
listen_addresses: '0.0.0.0:3000'

# Number of web server workers
worker_processes: 8

# Database configuration. This can also be a list of PG configs.
postgres:
  # Database connection string. You can use env vars too, for example:
  #   $DATABASE_URL
  #   ${DATABASE_URL:-postgresql://postgres@localhost/db}
  connection_string: 'postgresql://postgres@localhost:5432/db'

  # Same as PGSSLCERT for psql
  ssl_cert: './postgresql.crt'
  # Same as PGSSLKEY for psql
  ssl_key: './postgresql.key'
  # Same as PGSSLROOTCERT for psql
  ssl_root_cert: './root.crt'

  #  If a spatial table has SRID 0, then this SRID will be used as a fallback
  default_srid: 4326
  
  # Maximum connections pool size [default: 20]
  pool_size: 20

  # Limit the number of table geo features included in a tile. Unlimited by default.
  max_feature_count: 1000

  # Control the automatic generation of bounds for spatial tables [default: false]
  # If enabled, it will spend some time on startup to compute geometry bounds.
  disable_bounds: false

  # Enable automatic discovery of tables and functions. You may set this to `false` to disable.
  auto_publish:
    # Optionally limit to just these schemas
    from_schemas:
      - public
      - my_schema
    # Here we enable both tables and functions auto discovery.
    # You can also enable just one of them by not mentioning the other,
    # or setting it to false.  Setting one to true disables the other one as well.
    # E.g. `tables: false` enables just the functions auto-discovery.
    tables:
      # Optionally set a custom source ID based on the table name
      id_format: 'table.{schema}.{table}.{column}'
      # Add more schemas to the ones listed above
      from_schemas: my_other_schema
    functions:
      id_format: '{schema}.{function}'
  
  # Associative arrays of table sources
  tables:
    table_source_id:
      # ID of the MVT layer (optional, defaults to table name)
      layer_id: table_source
      
      # Table schema (required)
      schema: public
      
      # Table name (required)
      table: table_source
      
      # Geometry SRID (required)
      srid: 4326
      
      # Geometry column name (required)
      geometry_column: geom
      
      # Feature id column name
      id_column: ~
      
      # An integer specifying the minimum zoom level
      minzoom: 0
      
      # An integer specifying the maximum zoom level. MUST be >= minzoom
      maxzoom: 30
      
      # The maximum extent of available map tiles. Bounds MUST define an area
      # covered by all zoom levels. The bounds are represented in WGS:84
      # latitude and longitude values, in the order left, bottom, right, top.
      # Values may be integers or floating point numbers.
      bounds: [ -180.0, -90.0, 180.0, 90.0 ]
      
      # Tile extent in tile coordinate space
      extent: 4096
      
      # Buffer distance in tile coordinate space to optionally clip geometries
      buffer: 64
      
      # Boolean to control if geometries should be clipped or encoded as is
      clip_geom: true
      
      # Geometry type
      geometry_type: GEOMETRY
      
      # List of columns, that should be encoded as tile properties (required)
      properties:
        gid: int4
  
  # Associative arrays of function sources
  functions:
    function_source_id:
      # Schema name (required)
      schema: public
      
      # Function name (required)
      function: function_zxy_query
      
      # An integer specifying the minimum zoom level
      minzoom: 0
      
      # An integer specifying the maximum zoom level. MUST be >= minzoom
      maxzoom: 30
      
      # The maximum extent of available map tiles. Bounds MUST define an area
      # covered by all zoom levels. The bounds are represented in WGS:84
      # latitude and longitude values, in the order left, bottom, right, top.
      # Values may be integers or floating point numbers.
      bounds: [ -180.0, -90.0, 180.0, 90.0 ]

# Publish PMTiles files
pmtiles:
  paths:
    # scan this whole dir, matching all *.pmtiles files
    - /dir-path
    # specific pmtiles file will be published as pmtiles2 source
    - /path/to/pmtiles.pmtiles
  sources:
    # named source matching source name to a single file
    pm-src1: /path/to/pmtiles1.pmtiles
    
# Publish MBTiles files
mbtiles:
  paths:
    # scan this whole dir, matching all *.mbtiles files
    - /dir-path
    # specific mbtiles file will be published as mbtiles2 source
    - /path/to/mbtiles.mbtiles
  sources:
    # named source matching source name to a single file
    mb-src1: /path/to/mbtiles1.mbtiles
```