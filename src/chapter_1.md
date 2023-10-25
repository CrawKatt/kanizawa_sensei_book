# Modulo Main

```rust
/// Modulo prelude para importar todos los modulos principales y el Bot Schema
use crate::prelude::*;

/// dotenv para cargar las variables de entorno
use dotenv::dotenv;

/// Motor de base de datos SurrealDB
use surrealdb::engine::local::Mem;

/// Otros modulos del proyecto
mod commands;
mod enums;
mod error;
mod handlers;
mod prelude;
mod utils;

/// Modulos de la Base de Datos y Backup de la Base de Datos
use crate::utils::db::DB;
use crate::utils::load_data;

/// En Rust, la funcion main no puede ser async, 
/// por lo que se debe usar el macro #[tokio::main] 
/// de la libreria tokio
#[tokio::main]
async fn main() {
    // Inicializar la base de datos en memoria, en caso 
    // de que no se haya podido cargar la base de datos, se devolvera un panic!
    DB.connect::<Mem>(()).await.unwrap_or_else(|why| panic!("Ocurrio un error al conectar a la base de datos {why:#?}"));
    
    // Cargar los datos del backup.json a la base de datos al iniciar el Bot
    load_data().await.unwrap_or_else(|why| eprintln!("Ocurrio un error al cargar los datos: {why:#?}"));
    
    // Inicializar el logger
    pretty_env_logger::init();
    log::info!("Iniciando Bot...");
    
    // Cargar las variables de entorno
    dotenv().ok();

    // Inicializar el Bot Schema
    run().await;
}
```
