# Bot Schema
El Bot Schema del Bot se encuentra en el modulo prelude
    
```rust
use crate::handlers::chat_member_updates::{left_chat_member, new_chat_member};
use crate::handlers::command::common_command_handler;

use teloxide::{dispatching::UpdateFilterExt, dptree, prelude::Dispatcher};

use teloxide_core::adaptors::DefaultParseMode;
use teloxide_core::prelude::RequesterExt;
use teloxide_core::types::{ChatMemberUpdated, ParseMode, Update};

/// El tipo Bot es un alias para el tipo
/// DefaultParseMode<teloxide::Bot> y asi 
/// evitar escribir el tipo completo
pub type Bot = DefaultParseMode<teloxide::Bot>;

/// Funcion que inicializa el Bot Schema
pub async fn run() {
    // Inicializar el Bot
    let bot = teloxide::Bot::from_env().parse_mode(ParseMode::Html);
    
    // Inicializar el Handler que se encargara de manejar 
    // los mensajes, los botones, los updates, etc.
    let handler = dptree::entry()
        // Este inspect se encarga de imprimir
        // los Updates de Telegram (Usar solo para debug)
        // mantener comentado en produccion
        .inspect(|u: Update| {
            eprintln!("Update: {u:#?}");
        })
        // Este branch se encarga de manejar los comandos
        .branch(Update::filter_message().endpoint(common_command_handler))
        // Este branch se encarga de manejar los updates de los miembros del chat
        // si alguien se unio, o alguien se fue de un grupo
        .branch(
            Update::filter_chat_member()
                .branch(
                    dptree::filter(|m: ChatMemberUpdated| {
                        m.old_chat_member.is_left() && m.new_chat_member.is_present()
                    })
                        // El metodo endpoint se encarga de ejecutar
                        // la funcion encargada de manejar el update
                    .endpoint(new_chat_member),
                )
                .branch(
                    dptree::filter(|m: ChatMemberUpdated| {
                        m.old_chat_member.is_present() && m.new_chat_member.is_left()
                    })
                    .endpoint(left_chat_member),
                ),
        );

    // El Dispatcher se encarga de manejar el Bot y ejecutar los handlers
    Dispatcher::builder(bot, handler)
        .enable_ctrlc_handler()
        .build()
        .dispatch()
        .await;
}
```