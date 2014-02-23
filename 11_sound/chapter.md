#Sound

This chapter will demonstrate how to use the sound features and objects that you'll find in openFrameworks, as well as some techniques you can use to react to and generate sound.

`ofSoundPlayer` provides simple access to sound files, allowing you to easily load and play sounds, add sound effects to an app and extract some data about the file's sound as it's playing.

`ofSoundStream` gives you access to the computer's sound hardware, allowing you to programmatically generate your own sound as well as react to sound coming into your computer from something like a microphone or line-in jack.

- Mention ofSoundBuffer / ofSoundFile / ofSoundObject
- Quick and dirty play files w/ofSoundPlayer & beep boop `sin()` ofSoundStream examples to set the stage.

##Why -1 to 1?

In order to understand *why* openFrameworks chooses to represent sound as a continuous stream of `float` values ranging from -1 to 1, it'll be helpful to know how sound is created on a physical level.

*[ a minimal picture showing the mechanics of a speaker ]*

At the most basic level, a speaker consists of a cone and an electromagnet. The electromagnet pushes and pulls the cone to create vibrations in air pressure. These vibrations make their way to your ears, where they are interpreted as sound. When the electromagnet is off, the cone is simply "at rest", neither pulled in or pushed out.

[footnote] A basic microphone works much the same way: allowing air pressure to vibrate an object held in place by a magnet, thereby creating an electrical signal.

From the perspective of an openFrameworks app, it's not important what the sound hardware's specific voltages are. All that really matters is that the speaker cone is being driven between its "fully pushed out" and "fully pulled in" positions, which are represented as 1 and -1. [note: would be good to relate this to zach's bit about numbers between 0 and 1 in the animation chapter].

[footnote] Many other systems use an integer-based representation, moving between something like -65535 and +65535 with 0 still being the representation of "at rest". The Web Audio API provides an unsigned 8-bit representation, which ranges between 0 and 255 with 127 being "at rest". NOTE TO SELF DOUBLE CHECK THE SPECIFIC NUMBERS

A major way that sound differs from visual content is that there isn't really a "static" representation of sound. For example, if you were dealing with an OpenGL texture which represents 0 as "black" and 1 as "white", you could fill the texture with all 0s or all 1s and end up with a static image of "black" or "white" respectively. This is not the case with sound. If you were to create a sound buffer of all 0s, all 1s, all -1s, or any single number, they would all sound like exactly the same thing: nothing at all.

[footnote] Technically, you'd probably hear a pop right at the beginning as the speaker moves from the "at rest" position to whatever number your buffer is full of, but the remainder of your sound buffer would just be silence.

This is because what you actually hear is the *changes* in values over time. Any individual sample in a buffer doesn't really have a sound on its own. What you hear is the *difference* between the sample and the one before it. For instance, a sound's "loudness" isn't necessarily related to how "big" the individual numbers in a buffer are. A sine wave which osciallates between 0.9 and 1.0 is going to be much much quieter than one that osciallates between -0.5 and 0.5.

##Time Domain vs Frequency Domain

When representing sound as a continuous stream of values between -1 and 1, you're working with sound in what's known as the "Time Domain". This means that each value you're dealing with is referring to a specific moment in time. There is another way of representing sound which can be very helpful when you're using sound to drive something other aspect of your app. That representation is known as the "Frequency Domain".

*[ image of a waveform vs an FFT bar graph ]*

In the frequency domain, you'll be able to see how much of your input signal lies in various frequencies, split into various "bins" (see above image).

You can transform a signal in the time domain into the frequency domain by a ubiquitous algorithm called the Fast Fourier Transform. You can get an openFrameworks-ready implementation of the FFT (along with examples!) in either the ofxFFT or ofxFft addons (by Lukasz Karluk and Kyle McDonald respectively).

*[footnote explaining FFT vs DFT to avoid cluttering the previous paragraph up]*

You can also transform a signal from the frequency domain back to the time domain, using an Inverse Fast Fourier Transform (aka IFFT). This is less common, but there is an entire genre of audio synthesis called Additive Synthesis which is built around this principle (generating values in the frequency domain then running an IFFT on them to create synthesized sound).

- ofSoundStream gives you access to sound in the time domain.
- The time domain is useful for analysing general "loudness", as well as pitch detection ([counterintuitively](http://blog.bjornroche.com/2012/07/frequency-detection-using-fft-aka-pitch.html))
- Frequency domain is useful for isolating particular elements of a sound, such as instruments in a song. It is also useful for analyzing the character/timbre of a sound.

##Sound Files
- ofSoundPlayer is a tradeoff between ease-of-use and control. You get access to easy multiplay and pitch-shifted playback but lose precise control and access to the individual samples in the sound
- On the opposite end of the spectrum, ofSoundFile will allow you to extract an uncompressed ofSoundBuffer out of a file, allowing you access to the raw time domain signal.
- ofSoundPlayer provides access to the frequency domain content of the sounds being played in the form of `ofSoundGetSpectrum()`, but does not give access to the time domain (i.e. the -1 to 1 uncompressed samples)
- "Multiplay" allows you to have a file playing several times at different pitches simulatenously. Very handy for sound effects.

##Reacting to Live Audio

###RMS
One of the simplest ways to add audio-reactivity to your app is to calculate the RMS of incoming buffer of audio data. RMS stands for "root mean square" and is a pretty straightforward calculation that serves as a good approximation of "loudness" (much better than something like averaging the buffer or picking the maximum value). You can see RMS being calculated in the "audioInputExample".

*[ code snippet here ]*

###FFT
Running an FFT on your input audio will give you back a buffer of values representing the input's frequency content. A straight up FFT *won't* tell you which notes are present in a piece of music, but you will be able to use the data to take the input's sonic "texture" into account. For instance, the FFT data will let you know how much "bass" / "mid" / "treble" there is in the input at a pretty fine granulairty (a typical FFT used for realtime audio-reactive work will give you something like 512 to 4096 individual frequency bins to play with).

NOTE TO SELF/EDITORS: I definitely need to clean up the following paragraph. It's pretty crucial but I haven't found a way to get a succinct explanation of it yet.

When using the FFT to analyze music, you should keep in mind that the FFT's bins increment on a *linear* scale, whereas humans interpret frequency on a *logarithmic* scale. So, if you were to use an FFT to split an input signal into 512 bins, the lowest bins (probably bin 0 through bin 30 or so) will contain the bulk of the data, and the remaining bins will mostly just be high frequency content. If you were to isolate the sound on a bin-to-bin basis, you'd be able to easily tell the difference between the sound of bins 3 and 4, but bins 500 and 501 would probably sound exactly the same. Unless you had robot ears.

- Pitch detection
  - FFT -> Power -> IFFT Autocorrelation sort-of-hack
  - Zero crossings
- Onset detection
- Conversions to Mel scale, decibels

##Synthesizing Audio
- MIDI / OSC
- Attack Decay Sustain Release
- Working with external sound applications
- Overview of simple synthesis techniques (very high-level)

##Gotchas
### "Popping"
When starting or ending playback of synthesized audio, you should try to quickly fade in / out the buffer, instead of starting or stopping abruptly. If you start playing back a buffer that begins like `[1.0, 0.9, 0.8...]`, the first thing the speaker will do is jump from the "at rest" position of 0 immediately to 1.0. This is a *huge* jump, and will probably result in a "pop" that's quite a bit louder than you were expecting (based on your computer's current volume).

Usually, fading in / out over the course of about 30ms is enough to eliminate these sorts of pops.

If you're getting pops in the middle of your playback, you can diagnose it by trying to find reasons why the sound might be very briefly cutting out (i.e. jumping to 0, resulting in a pop if the waveform was previously at a non-zero value).

### "Clipping" / Distortion
If your samples begin to exceed the range of -1 to 1, you'll likely start to hear what's known as "clipping", which generally sounds like a grating, unpleasant distortion. Some audio hardware will handle this gracefully by allowing you a bit of leeway outside of the -1 to 1 range, but others will "clip" your buffers.

*[ clipped waveform image ]*

Assuming this isn't your intent, you can generally blame clipping on a misbehaving addition or subtraction in your code. A multiplication of any two numbers between -1 and 1 will always result in another number between -1 and 1.

If you *want* distortion, it's much more common to use a waveshaping algorithm [todo: link].

  - Sample rates, Nyquist, aliasing
  - Latency