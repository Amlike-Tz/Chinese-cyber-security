## SDR radio and doorbell replay experiment (hackrf one)

## Preface

Hardware environment to prepare:hackrf, antenna(40-860MHz), doorbell.

Software environment:gqrx, ubuntu18.04, audacity, hackrf tools

Here on the construction of software tools will not repeat, build the process of the students have questions can find me to explore, I am also in the learning stage, welcome to exchange!

So back to business, HackRF One is a broadband software-defined radio half-duplex transceiver created and manufactured by Great Scott Gadgets. Notice it's half-duplex, what is half-duplex? Half-duplex is data transmission means that data can be transmitted in both directions of a signal carrier, but not at the same time. That is to say, you can't do both receive and transmit operations at the same time, which is a bit of a cock-up, but that's not a big deal in the face of the high price of blade as well as usrp, hahahaha.

 

## SDR listening to FM radio

Below is my hackrf, it's the PORTAPACK version so it doesn't look quite the same.

! [img](https://p3.ssl.qhimg.com/t01ca38d2eacd93d4cb.png)

First plug in the hackrf, because my ubuntu is installed in vmware, so the device is plugged into the physical machine and connects to the virtual machine, then in the terminal type hackrf_info or SoapySDRUtil -find, if it returns the information in the picture below, then the device has been connected successfully!

! [img](https://p5.ssl.qhimg.com/t0123d62260d45fcaab.png)

Then continue to type gqrx -r in the terminal to open the tool, pay attention to the -r parameter, because entering gqrx directly may result in an exception, when the following window pops up, select our device in Device, other parameters do not move, and then click ok to enter the tool.

! [img](https://p5.ssl.qhimg.com/t0190945a2b3ebc345a.png)

After opening, the interface is as below.

! [img](https://p3.ssl.qhimg.com/t01120c472ebe1366de.png)

On the left side is the spectrum map, on the right side we need to pay attention to the Mode, AGC, and Squelch. adjust the Mode to WFM (mono), which means Wideband FM (mono), so that the audio quality is higher compared to Narrow FM. Click on the triangle in the upper left corner to start receiving radio signals.

Adjust the Mode to WFM (mono).
AGC is adjusted to Fast.
Squelch (squelch) is adjusted, and stops when you hear a squealing current.

Click input controls at the bottom right.
Drag the IF (Intermediate Frequency) to the right to 32dB.
BB (Baseband FM) to 52dB.

Usually, the frequency band of FM broadcast is 80-110MHZ, we will adjust the frequency band to 100MHZ, and found that there are already some peaks, generally speaking, these are the broadcast band.

! [img](https://p2.ssl.qhimg.com/t01c51a12c092d0b1de.png)

I here in the environment adjusted to 107.600.000hz when you can hear the radio sound clearly, by observing the various peaks can basically find the broadcast band.

Note:The frequency band can be adjusted directly by sliding the wheel.RF is the radio frequency gain, IF is the intermediate frequency gain, BB is the baseband gain. The relationship between them is Receive: RF-IF-BB, that is, the RF high-frequency signal is converted to IF IF and then to BB baseband processing, after two frequency conversion. This operation is also to reduce interference and ensure normal and stable operation.

 

## Wireless Doorbell Replay Experiment

Regarding the principle of doorbell replay experiment, it is just to receive and record the signal sent by the doorbell button through hackrf and then replay it, so that the doorbell receives the signal sent and then rings.

The first thing we need to do is to determine the transmission frequency and reception frequency of the doorbell. By purchasing the doorbell, we can learn that the transmit frequency of the doorbell button is 433.92MHZ and the receive frequency is 433.92MHZ.

! [image](https://user-images.githubusercontent.com/37897216/233993050-e67e0798-313d-46c3-b7b4-82159f339869.png)

After confirming the frequency, we always have to verify that it is not. Then it's off to turn on the gqrx, adjust the frequency to 433.92MHZ and press the doorbell.

! [img](https://p0.ssl.qhimg.com/t011febb7b79d0adf14.png)

When the button is pressed, the spectrogram wave crest rises and then disappears. From the yellow portion below the corresponding frequency you can see that energy is generated at that frequency, interrupt the button and find that the generation of energy is interrupted with the generation of energy. Determine the frequency of the transmitter to be 433.92 MHZ.

Enter the command.

``
hackrf_transfer -r door.raw -f 433920000 -g 16 -l 32 -a 1 -s 8000000 -b 4000000
```

Then it is executed and hackrf starts receiving the recording signal. Press the button at this point, here I pressed it twice intermittently for the sake of the display. Then Ctrl+c to stop recording.

The meaning of the command parameters is as follows.

-r <filename> # Receive data into a file.
[-f set_freq_hz]#Set frequency in Hz
[-g gain_db]#RX VGA (baseband) gain, 0-62dB, 2dB steps
[-l gain_db]#Set low noise amplifier, 0-40dB, 8dB steps
[-a set_amp]#RX/TX RF amplifier 1=enable, 0=disable.
[-s sample_rate_hz]#Set sample rate in Hz (8/10 / 12.5 / 16 / 20MHz)
[-b baseband_filter_bw_hz]#Set the baseband filter bandwidth (MHz).

! [img](https://p0.ssl.qhimg.com/t015ebd2f7b8240b363.png)

After successful recording we open audacity to view the recorded raw file (door.raw). The values in the window that pops up when you open it do not need to be adjusted. Just click on File and then Import. As shown in the figure we can see that there are two parts, which is that we record two ringing signals, that we have recorded successfully, there may be small partners will ask why use this tool to see this file? Is not superfluous, this we say below.

! [img](https://p0.ssl.qhimg.com/t01462ecc2e04e26b85.png)

After determining that there are no problems with the recording file, we continue to build the Send Signal command.

Enter the command.

``
hackrf_transfer -t door.raw -f 433920000 -x 47 -a 1 -s 8000000 -b 4000000
```

The command parameters are as follows.
-t <filename> # Transfer data from file.
[-x gain_db]#Set TX vga gain, 0-47dB in 1dB steps.

Other parameters as above are not listed.

! [img](https://p0.ssl.qhimg.com/t014b9a9f534952283a.png)

The wireless doorbell plays successfully and the experiment is over.

Here may appear the following problems:

1, because previously introduced hackrf is a half-duplex board, so sometimes can not be normal recording can try to restart the device can be solved. For example.
`hackrf_open() failed:Resource busy (-1000)` is the problem.

2, also need to pay attention to, -f specify the frequency, above we said that the receiver and sender frequency is 433.92MHZ, so we need to pay attention to build the receive or send commands in their specific environment to adjust.

3, about the value of some of the parameters such as -s, -b, -x, -g, -l, etc. These values are not fixed can also be adjusted according to their own specific environment, the above parameter values are only for reference.

3, -s specifies the sampling rate, the higher the sampling frequency, that is, the shorter the sampling interval, the more sample data obtained by the computer within a unit of time, the more accurate representation of the signal waveform. However, this does not mean that you can adjust it at will, you still need to pay attention to the conditions of the frequency value.

4, why use audacity to view the recording file, because in the operation, I found that sometimes may not record successfully, such as the end of the recording and then replay the doorbell does not respond, then we need to be in the tool through the magnifying glass to see whether the successful recording, if the signal in the figure above that should be a successful recording.

5, pay attention to the test as far as possible away from high interference equipment such as freezers, high-power transmitters to avoid affecting the experimental effect, thus appearing unsatisfactory results.

 

## Postscript

The above is hackrf listening to FM radio and replay the experiment process, there are questions students can contact me to learn together. This article is more inclined to the basics, suitable for students who have just started, I hope to have read this article can be gained for you.

I wish you all the best of luck and success!