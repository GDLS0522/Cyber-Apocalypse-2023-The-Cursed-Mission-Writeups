# Somewhat Linear write up by Om Honrao

## Challenge Description
```
Deep in the alien cave system, you find a strange device. It seems to be some sort of communication cipherer, but only a couple of recordings are still intact. Can you figure out what the aliens were trying to say? The flag is all lower case
```

## Attachments
```bash
- rev_somewhat_linear.zip
    ./impulse_response.wav
    ./shuffled_flag.wav
    ./input_generator.py
```

## Solution
The only hard chall done by my team in Reversing by me and was interesting enough. I started off with [input_henerator.py](./input_generator.py) here is what i understood from the code:
```
This code imports the NumPy and SoundFile libraries and reads an audio file named "flag.wav" using the sf.read() function. Then, it randomly shuffles the frequencies of the audio signal using NumPy's Fast Fourier Transform (FFT) functions.

The np.fft.rfftfreq() function computes the frequencies corresponding to the positive frequencies of the FFT output. The length of the FFT is determined by the length of the audio signal (len(flag)) and the sampling rate (rate).

The np.random.uniform() function generates a random array of uniformly distributed values between -10 and 10, which will be used to shuffle the amplitudes of the audio signal.

The filter() function takes an audio sample and applies the frequency shuffling by performing the FFT of the input signal, multiplying the shuffled frequency response, and then performing the inverse FFT. The resulting audio signal will have the same duration as the input signal but with a different frequency distribution.

To create an impulse response, the code generates an array of zeros named "impulse" with the same length as the audio signal and sets the first element to 1. This array is then passed to the filter() function to obtain the shuffled impulse response, which is saved to a new audio file named "impulse_response.wav" using the sf.write() function.

Finally, the filter() function is called again, this time with the original audio signal "flag" as the input. The shuffled audio signal is then saved to a new audio file named "shuffled_flag.wav" using the sf.write() function. The resulting audio file will have the same duration and format as the original file but with a different frequency distribution due to the frequency shuffling.
```
Hmm i know this thing might go over the brain for someone not familiar to reversing but reading it carefully you can understand. So i made a exploit script to reverse the process and get the flag. Here is the script:

```python
import numpy as np
import soundfile as sf

shuffled_flag, rate = sf.read('shuffled_flag.wav')
impulse_response, _ = sf.read('impulse_response.wav')

# Take the FFT of the impulse response
fft_ir = np.fft.rfft(impulse_response)

# Divide the FFT of the shuffled audio file by the FFT of the impulse response
fft_shuffled = np.fft.rfft(shuffled_flag)
fft_orig = fft_shuffled / fft_ir

# Inverse FFT the result to get the original audio file
orig = np.fft.irfft(fft_orig)

# Save the recovered audio file
sf.write('flag.wav', orig, rate)
```
Here's what the exp code does:
```T
his code reads in the shuffled audio file "shuffled_flag.wav" and the impulse response file "impulse_response.wav" using the SoundFile library. Then, it applies inverse filtering to the shuffled audio file using the impulse response.

First, the FFT of the impulse response is computed using the np.fft.rfft() function, which returns an array of complex values representing the positive frequencies of the FFT output.

Next, the FFT of the shuffled audio file is computed using the same function, and then divided element-wise by the FFT of the impulse response to obtain the FFT of the original audio signal.

Finally, the inverse FFT is applied to the result using the np.fft.irfft() function, which returns a real-valued array representing the recovered original audio signal.

The recovered audio signal is then saved to a new audio file named "new.wav" using the sf.write() function.

This process of dividing the FFT of the shuffled audio signal by the FFT of the impulse response to obtain the FFT of the original audio signal is called deconvolution or inverse filtering. In this case, it is used to recover the original audio signal that was shuffled by the frequency response in the previous code.
```
 This results the following [flag.wav](./recovered_flag.wav) file. Opening it and hearing we might think it's not the flag but it is the flag the person says words like Bravo, Siara, etc just consider the first letters of all those words and yes thats the flag!

## Flag
```
HTB{th1s_w@s_l0w_eff0rt}   
```

## Contact info

This challenge was solved by Om Honrao (Discord ID: Inv1s1bl3#7047). If you have any questions or feedback, feel free to reach out to me.