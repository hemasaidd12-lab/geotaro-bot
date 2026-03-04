const { default: makeWASocket, useMultiFileAuthState } = require("@whiskeysockets/baileys")

let points = {}
const badWords = ["كلمه_شتيمه1","كلمه_شتيمه2"] // استبدل بكلمات الشتايم اللي عايز تمنعها

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState("auth")
    const sock = makeWASocket({ auth: state })

    sock.ev.on("creds.update", saveCreds)

    sock.ev.on("messages.upsert", async ({ messages }) => {
        const m = messages[0]
        if (!m.message || !m.key.remoteJid.endsWith("@g.us")) return

        const sender = m.key.participant
        const text = m.message.conversation || ""

        // مايردش غير لو الرسالة بدأت بـ "."
        if (!text.startsWith(".")) return

        // أوامر
        const args = text.slice(1).split(" ")
        const command = args[0]

        if (command === "اوامر") {
            await sock.sendMessage(m.key.remoteJid, { text: "📌 الاوامر:\n.اوامر\n.نقاط\n.لعب" })
        }

        if (command === "نقاط") {
            if (!points[sender]) points[sender] = 0
            await sock.sendMessage(m.key.remoteJid, { text: `⭐ نقاطك: ${points[sender]}` })
        }

        if (command === "لعب") {
            if (!points[sender]) points[sender] = 0
            points[sender] += 1
            await sock.sendMessage(m.key.remoteJid, { text: "🎮 كسبت نقطة!" })
        }

        // منع الشتايم
        badWords.forEach(word => {
            if (text.includes(word)) {
                sock.sendMessage(m.key.remoteJid, { text: "🚫 ممنوع الشتايم" })
            }
        })
    })
}

startBot()# geotaro-bot
