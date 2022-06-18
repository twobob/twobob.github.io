## Doing some python WPM analysis 

Since I generally watch Youtube video at 200% speed I was interested when a certain YouTuber "James Sharman"'s videos caused me to watch at less than this speed.

He was purely intelligible and not speaking seemingly "fast", could it just be the information density?

So I set about writing a (mean average) Words Per Minute app in python to check the numbers. 

---

### Getting it done

#### Using the libs

```python
from speech_recognition import *
from os import walk, path
import glob
import wave
import nltk
import contextlib
from pydub import AudioSegment
from pydub.silence import split_on_silence
import subprocess
```

This nets us the ability to slice, load, refactor and ananlyse audio with a variety of tools.

#### Google needs it piecemeal

Turns out that the Google service requires for the data to be in short chunks (and the sphinx engine did not do well on James' voice) so we split in on sensible silences and send those chunks, then gather them back up again as a completed text corpus.

```python
def split(filepath):
	sound = AudioSegment.from_file(filepath)
	chunks = split_on_silence(
	sound,
		min_silence_len = 500,
		silence_thresh = sound.dBFS - 16,
		keep_silence = 250, # optional
		)
	return chunks

def recognize_chunks(chunk):
	# Create a silence chunk that's 0.5 seconds (or 500 ms) long for padding.
	silence_chunk = AudioSegment.silent(duration=500)
	# Add the padding chunk to beginning and end of the entire chunk.
	audio_chunk = silence_chunk + chunk + silence_chunk
	# Export the audio chunk with new bitrate.
	audio_chunk.export(".//tempchunk.wav",bitrate = "44100",format = "wav")
	r = Recognizer()
	inp = AudioFile(".//tempchunk.wav")
	with inp as source:
		audio = r.record(source)
		try:
			all_words = r.recognize_google(audio)
			return all_words
		except:
			return "No speech detected"
            
# and later call that if the file is quite long...           
    
    if duration > 120:
					print(f"total words so far in array from chunks: ", end ='' )
					chunks = split(filename)
					output_chunks = [chunks[0]]
					for chunk in chunks[1:]:
						if len(output_chunks[-1]) < target_length:
							output_chunks[-1] += chunk
						else:
							output_chunks.append(chunk)

					for chunk in output_chunks:
						tempwords = recognize_chunks(chunk)
						if tempwords != "No speech detected":
							words += tempwords
						#print(tempwords)
						print(len(words.split(' '), end=''), "total words so far in array from chunks")
				else:
					words = recognize(filename)
    
```

Once we have the texts of the various files; classification is relatively simple

```python
if not path.exists(filename+".distribution.txt"):
				with open(filename+".distribution.txt", "w") as o:
					# create a frequency distribution
					fdist = nltk.FreqDist(all_words[filename].split(' '))
					# print the top 150 most spoken words
					o.write("word frequency report for "+ filename.replace('wav','')+"\n\n")
					for w, count in fdist.most_common(150):
						if not w.isspace():
							report = w.ljust(20) + str(count) + " \n"
							o.write(report)
		else:
			print(filename, 'skipped')
```

and the mean average is obvisouly just the amount of words in total over the duration. 
we can grab the duration like so:

```python

with contextlib.closing(wave.open(filename,'r')) as f:
					frames = f.getnframes()
					rate = f.getframerate()
					duration = frames / float(rate)
```

The other details are available in the code on Gist https://gist.github.com/twobob/e772c3b207fc8cfca5c2e4008aefceb0

I shall publish the results of the video analysis, sufficed to say that whilst James does indeed maintain a healthy 200 - 250ish WPM it is the information density and new ideas that make me slow the videos. Not his talking :)

excerpt of one playlist:

```
.\1 - Introduction - VGA from Scratch - Part 1              : mins:22 secs:52 total words:3360 words per min:305.45454545454544
.\2 - Sync - VGA from Scratch - Part 2                      : mins:31 secs:43 total words:3867 words per min:249.48387096774192
.\3 - Framebuffer - VGA from Scratch - Part 3               : mins:46 secs:35 total words:6437 words per min:279.8695652173913
.\4 - Hardware Scrolling - VGA from Scratch - Part 4        : mins:31 secs:3  total words:3957 words per min:255.29032258064515
.\5 - Beam Racing - VGA from Scratch - Part 5               : mins:16 secs:48 total words:2445 words per min:305.625
.\6 - PCB Planning - VGA from Scratch - Part 6              : mins:19 secs:17 total words:2431 words per min:255.89473684210526
.\7 - Sync PCB - VGA from Scratch - Part 7                  : mins:28 secs:52 total words:2424 words per min:173.14285714285714
.\8 - Interface PCB - VGA from Scratch - Part 8             : mins:33 secs:18 total words:3205 words per min:194.24242424242425
.\9 - Tilemap (Framebuffer) PCB - VGA from Scratch - Part 9 : mins:47 secs:12 total words:3665 words per min:155.95744680851064
```
