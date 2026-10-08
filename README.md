# openai-elevenlabs-automation
Automated AI pipeline using Next.js to generate scripts via OpenAI and convert them to speech via ElevenLabs API
```typescript
import { generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
import fs from 'fs';

// 1. Generate Structured Text via OpenAI
export async function generateContent(topic: string) {
  const { object } = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      text: z.string().describe("The text to be spoken, optimized for TTS")
    }),
    prompt: `Generate a short script about: ${topic}`
  });
  return object.text;
}

// 2. Convert to Audio via ElevenLabs API
export async function generateAudio(text: string, voiceId: string, apiKey: string) {
  const response = await fetch(`https://api.elevenlabs.io/v1/text-to-speech/${voiceId}`, {
    method: 'POST',
    headers: {
      'Accept': 'audio/mpeg',
      'Content-Type': 'application/json',
      'xi-api-key': apiKey
    },
    body: JSON.stringify({
      text,
      model_id: 'eleven_multilingual_v2',
      voice_settings: { stability: 0.4, similarity_boost: 0.8 }
    })
  });

  if (!response.ok) throw new Error('ElevenLabs API Error');
  
  const buffer = Buffer.from(await response.arrayBuffer());
  fs.writeFileSync('./output.mp3', buffer);
  return './output.mp3';
}
