module.exports = async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, x-app-pin');

  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const appPin = process.env.APP_PIN;
  const geminiApiKey = process.env.GEMINI_API_KEY;

  if (!geminiApiKey) {
    return res.status(500).json({ error: 'GEMINI_API_KEY belum diatur di Vercel Environment Variables.' });
  }

  const body = typeof req.body === 'string' ? JSON.parse(req.body) : (req.body || {});
  const userPin = req.headers['x-app-pin'] || body.pin;

  // Validasi PIN akses aplikasi
  if (appPin && String(userPin) !== String(appPin)) {
    return res.status(401).json({ error: 'PIN salah atau tidak valid.' });
  }

  // Endpoint khusus cek PIN saat login pertama kali
  if (body.action === 'verify_pin') {
    return res.status(200).json({ ok: true });
  }

  try {
    const model = body.model || 'gemini-2.5-flash';
    const payload = {
      contents: body.contents || [],
    };

    if (body.systemInstruction) {
      payload.systemInstruction = body.systemInstruction;
    }

    if (body.generationConfig) {
      payload.generationConfig = body.generationConfig;
    }

    const response = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${geminiApiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      }
    );

    const data = await response.json();

    if (!response.ok) {
      return res.status(response.status).json({
        error: data.error?.message || 'Gagal menghubungi Gemini API.',
      });
    }

    return res.status(200).json(data);
  } catch (error) {
    return res.status(500).json({ error: error.message || 'Terjadi kesalahan pada server.' });
  }
};