#const {
  default: makeWASocket,
  useMultiFileAuthState,
  DisconnectReason
} = require("@whiskeysockets/baileys");

const P = require("qrcode-terminal");

async function startBot() {
  const { state, saveCreds } = await useMultiFileAuthState("auth_info");

  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: false
  });

  sock.ev.on("connection.update", ({ connection, qr }) => {
    if (qr) {
      P.generate(qr, { small: true });
    }

    if (connection === "close") {
      startBot();
    }

    if (connection === "open") {
      console.log("✅ WhatsApp Bot Connected!");
    }
  });

  sock.ev.on("creds.update", saveCreds);

  sock.ev.on("messages.upsert", async ({ messages }) => {
    const msg = messages[0];

    if (!msg.message) return;

    const text =
      msg.message.conversation ||
      msg.message.extendedTextMessage?.text;

    const from = msg.key.remoteJid;

    if (text === ".menu") {
      await sock.sendMessage(from, {
        text: `🤖 *Bot Menu*

.menu - Show this menu
.ping - Check bot status
.owner - Owner information`
      });
    }

    if (text === ".ping") {
      await sock.sendMessage(from, {
        text: "🏓 Pong! Bot is online."
      });
    }

    if (text === ".owner") {
      await sock.sendMessage(from, {
        text: "👤 Owner: Your Name"
      });
    }
  });
}

startBot();
