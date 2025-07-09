# Nohara-Rin-Bot-v1
Bot WhatsApp Nohara Rin 🌸
https://github.com/Nohara-Rin/Nohara-Rin-Bot-v1
{
  "name": "nohara-rin-wa",
  "version": "1.0.0",
  "description": "Bot WhatsApp Nohara Rin 🌸",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@adiwajshing/baileys": "^4.4.0",
    "qrcode-terminal": "^0.12.0"
  }
}
index.js
const { default: makeWASocket, useSingleFileAuthState } = require("@adiwajshing/baileys");
const qrcode = require("qrcode-terminal");
const { Boom } = require("@hapi/boom");

const { state, saveState } = useSingleFileAuthState("./auth_info.json");

const startBot = async () => {
  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true,
  });

  sock.ev.on("creds.update", saveState);

  sock.ev.on("connection.update", (update) => {
    const { connection, lastDisconnect } = update;
    if (connection === "close") {
      const shouldReconnect = lastDisconnect.error?.output?.statusCode !== DisconnectReason.loggedOut;
      console.log("Koneksi terputus. Reconnect?", shouldReconnect);
      if (shouldReconnect) {
        startBot();
      }
    } else if (connection === "open") {
      console.log("Nohara Rin Bot v1 aktif dan siap!");
    }
  });

  sock.ev.on("messages.upsert", async ({ messages }) => {
    const msg = messages[0];
    if (!msg.message || msg.key.fromMe) return;

    const from = msg.key.remoteJid;
    const body = msg.message.conversation || msg.message.extendedTextMessage?.text || "";

    if (body === ".menu") {
      await sock.sendMessage(from, { text:
`*🌸 Nohara Rin Bot v1 Menu 🌸*
1. .halo – Sapaan
2. .pantun – Pantun random
3. .waktu – Jam saat ini
4. .menu – Menampilkan menu`
      });
    } else if (body === ".halo") {
      await sock.sendMessage(from, { text: "Hai juga! Aku Nohara Rin Bot v1 🌸" });
    } else if (body === ".pantun") {
      let pantun = [
        "Pergi ke pasar beli semangka\nLihat kamu, hati berbunga 🌸",
        "Jalan-jalan ke Surabaya\nLiat kamu bikin hati bahagia 😳",
        "Naik motor ke kota lama\nKamu cakep tapi banyak drama 😜"
      ];
      let pilih = pantun[Math.floor(Math.random() * pantun.length)];
      await sock.sendMessage(from, { text: pilih });
    } else if (body === ".waktu") {
      const now = new Date().toLocaleString("id-ID", { timeZone: "Asia/Jakarta" });
      await sock.sendMessage(from, { text: `⏰ Sekarang: ${now}` });
    }
  });
};

startBot();
