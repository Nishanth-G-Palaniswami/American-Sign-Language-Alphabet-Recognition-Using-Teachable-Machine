# English to ASL

An interactive English to American Sign Language (ASL) translator. A local LLM converts an English sentence into ASL gloss tokens, each token is matched to a recorded sign video, and OpenCV plays the clips in sequence. When no exact clip exists, spaCy word-vector similarity picks the closest available sign. Everything runs on your machine.

This repository is the English-to-ASL video prototype. The IEEE ICCCNT 2024 paper "American Sign Language Alphabet Recognition Using Teachable Machine" is a separate project (alphabet recognition from camera input) and its code is not in this repo.

## How it works

1. **Input.** A PySimpleGUI window takes an English sentence.
2. **Gloss tokens.** The sentence and a few-shot prompt go to a local LLM served by LM Studio on port 1234. Llama 2 7B was the model when this was built; any chat model behind LM Studio's OpenAI-compatible endpoint works.
3. **Playback.** Each token maps to `src/Videos/<token>.mp4` and OpenCV plays the clips in order. Press `q` during a clip to skip to the next one.
4. **Fallback.** If a token has no clip, `similarity.py` uses spaCy `en_core_web_md` vectors to pick the nearest available sign above a 0.5 similarity threshold.

## Requirements

- Python 3.8+
- [LM Studio](https://lmstudio.ai/) with a chat model loaded and its local server running on `http://localhost:1234`
- spaCy English model: `python -m spacy download en_core_web_md`

## Setup

```bash
git clone https://github.com/Nishanth-G-Palaniswami/english-to-asl.git
cd english-to-asl
pip install opencv-python PySimpleGUI numpy openai spacy
python -m spacy download en_core_web_md
```

## Run

```bash
cd src
python videoPlayer.py
```

Type a sentence, click **Play Videos**, and the signs play in sequence.

## Vocabulary

About 2,000 sign clips live under `src/Videos/`, one per gloss token and named by the token (for example `thank you.mp4`, `hot dog.mp4`). The filenames are the vocabulary; list the folder to see it.

## Layout

```
src/
  videoPlayer.py     entry point: GUI, token sequencing, OpenCV playback
  APICall.py         LLM request with the few-shot prompt
  similarity.py      spaCy similarity fallback
  Videos/            sign clips
helpers/             scratch scripts and tests
EnglishToASLDocs/    prompt notes
```

## Functions

- `main()`: entry point.
- `getInput() -> str`: PySimpleGUI window with a text field and Play / Exit buttons; returns the sentence.
- `Sequence(inp: list, action: str) -> None`: maps tokens to filenames and calls `playVideo()` for each.
- `playVideo(inputFile: str, action: str) -> None`: plays one clip from `Videos/` with OpenCV.
- `APICall(QUERY: str) -> list`: sends the prompt and query to the LLM, returns the token list.
- `similaritySearch(token: str) -> list`: spaCy similarity search, returns `[TOKEN, SCORE, INDEX]` candidates.

## Prompt

The LLM receives this system prompt, verbatim:

```plaintext
You are a helper that converts English words into ASL tokens.
You will be given an English sentence. You must convert it into tokens.
The following are a few examples of how to do the same.
Example 1
input : "How are you"
output : "How, are, you"
Example 2
input : "I am hungry"
output : "I, hungry"
Example 3
input : "Water the plants"
output : "water, plant"
Example 4
input : "I am feeling hot"
output : "I, feel, hot"
Example 5
input : "Invite your friends to the party"
output : "invite, friend, to, party"
Example 6
input : "Can I have mushroom and pumpkin soup"
output : "I, want, mushroom, pumpkin, soup"
Example 7
input : "Let us plan for a movie tonight"
output : "Let, us, plan, for, a, movie, tonight"
Example 8
input : "Would you like a coffee"
output "how, about, a, coffee"
Example 9
input : "Plan your day ahead for better result"
output : "Plan, your, day, ahead, for, better, result"
Example 10
input : "Car race is my favourite sport"
output : "I, love, car, race"
Using the above examples as reference maintian the meaning and context of the sentence, for any given user input generate the ASL Tokens ONLY separated by commas.
```

Followed by the query:

```plaintext
input text is " {USER_INPUT} "
convert this into ASL tokens
```

## Limitations

- Gloss generation is prompt-driven. Word order and dropped function words follow the few-shot examples, not a grammar model, so output quality varies with the LLM.
- The similarity fallback is approximate and can choose a semantically near but wrong sign.
- Playback is sequential clip concatenation, not continuous signing.
- This is an educational prototype, not a substitute for a certified interpreter.

## Troubleshooting

- **LM Studio not reachable:** confirm the local server is started and listening on port 1234.
- **Missing spaCy model:** run `python -m spacy download en_core_web_md`.
- **Clip not found:** the similarity fallback runs automatically; the console shows the substitute it chose.

## License

MIT, see [LICENSE](LICENSE).
