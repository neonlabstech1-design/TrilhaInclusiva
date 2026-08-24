// Função serverless do Vercel — roda no servidor, nunca no navegador.
// A chave da OpenRouter fica só aqui, lida de uma variável de ambiente,
// então nenhum professor precisa (nem consegue) ver ou configurar a chave.
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    res.status(405).json({ error: 'Método não permitido.' });
    return;
  }

  const apiKey = process.env.OPENROUTER_API_KEY;
  if (!apiKey) {
    res.status(500).json({ error: 'OPENROUTER_API_KEY não configurada no servidor. Configure em Vercel → Project Settings → Environment Variables.' });
    return;
  }

  try {
    const { instructions, imageBase64, imageMime, model } = req.body || {};
    if (!instructions) {
      res.status(400).json({ error: 'Faltam as instruções para gerar a prova.' });
      return;
    }

    const content = [{ type: 'text', text: instructions }];
    if (imageBase64) {
      content.unshift({ type: 'image_url', image_url: { url: `data:${imageMime || 'image/png'};base64,${imageBase64}` } });
    }

    const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + apiKey,
        'HTTP-Referer': 'https://trilha-inclusiva.vercel.app',
        'X-Title': 'Trilha Inclusiva',
      },
      body: JSON.stringify({
        model: model || 'anthropic/claude-sonnet-4.5',
        max_tokens: 2000,
        messages: [{ role: 'user', content }],
      }),
    });

    const data = await response.json();
    if (data.error) {
      res.status(500).json({ error: data.error.message || 'Erro na geração da prova.' });
      return;
    }

    const msg = data.choices && data.choices[0] && data.choices[0].message;
    let text = '';
    if (typeof msg?.content === 'string') text = msg.content;
    else if (Array.isArray(msg?.content)) text = msg.content.map(b => b.text || '').filter(Boolean).join('\n');

    if (!text) {
      res.status(500).json({ error: 'A IA não retornou conteúdo.' });
      return;
    }

    res.status(200).json({ text });
  } catch (e) {
    res.status(500).json({ error: e.message || String(e) });
  }
}
