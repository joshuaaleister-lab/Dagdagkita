// Free, high-quality voiceovers via Microsoft Edge TTS (edge-tts).
// No API key, no credits, no per-use cost. Deployed automatically by Vercel at /api/tts
const { MsEdgeTTS, OUTPUT_FORMAT } = require("msedge-tts");

module.exports = async (req, res) => {
  // Allow the page to call this
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");
  if (req.method === "OPTIONS") return res.status(200).end();
  if (req.method !== "POST") return res.status(405).json({ error: "POST only" });

  try {
    let body = req.body;
    if (typeof body === "string") { try { body = JSON.parse(body); } catch (_) { body = {}; } }
    body = body || {};
    const text = String(body.text || "").slice(0, 5000).trim();
    const voice = String(body.voice || "en-US-AriaNeural");
    if (!text) return res.status(400).json({ error: "No text provided" });

    const tts = new MsEdgeTTS();
    await tts.setMetadata(voice, OUTPUT_FORMAT.AUDIO_24KHZ_48KBITRATE_MONO_MP3);
    const { audioStream } = tts.toStream(text);

    const chunks = [];
    await new Promise((resolve, reject) => {
      const done = () => resolve();
      audioStream.on("data", (c) => chunks.push(c));
      audioStream.on("end", done);
      audioStream.on("close", done);
      audioStream.on("error", reject);
      setTimeout(() => reject(new Error("TTS timeout")), 25000);
    });

    const buf = Buffer.concat(chunks);
    if (!buf.length) return res.status(502).json({ error: "Empty audio from TTS engine" });

    res.setHeader("Content-Type", "audio/mpeg");
    res.setHeader("Cache-Control", "no-store");
    return res.status(200).send(buf);
  } catch (e) {
    return res.status(500).json({ error: String((e && e.message) || e) });
  }
};
