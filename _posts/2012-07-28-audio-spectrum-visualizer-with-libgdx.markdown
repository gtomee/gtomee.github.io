---
layout: post
title: Audio spectrum visualizer with libGDX
categories:
- Audio Spectrum libGDX
- Labs
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  _oembed_c040ab108ba36845e977c64966886b52: <iframe width="550" height="309" src="http://www.youtube.com/embed/5cN1VzZXcdo?feature=oembed"
    frameborder="0" allowfullscreen></iframe>
---
Hi,

Today, we're going to do some music. This experiment will be playable as a desktop application and an Android application. Unfortunately, this won't work in the browser because the libGDX HTML5 backend doesn't handle audio decoders.

Here is a video of what it is possible to do using libGDX audio tools, and what we'll do next:

<iframe width="640" height="360" src="//www.youtube.com/embed/5cN1VzZXcdo?rel=0" frameborder="0" allowfullscreen></iframe>

The entire source code is on our github : <a href="https://github.com/gtomee/AudioSpectrumGDX">https://github.com/gtomee/AudioSpectrumGDX</a>. However we only put the source file, libGDX library and images resources. We didn't put any audio file because you can use your own and set it in the FILE string (see below).
<h2>Getting started with libGDX</h2>
First, you need to know how to use libGDX. If you are familiar with this library, it's fine ! Otherwise you can have a look <a href="http://libgdx.badlogicgames.com/">here</a> to learn how to use it, and <a href="http://code.google.com/p/libgdx/wiki/TableOfContents">here</a> for some tutorials. LibGDX is a great engine for building games for Android platform, java desktop and also HTML5 (WebGL). We recommend you to use the gdx-setup-ui tool to help you to setup a new project very easily !
<h2>Setting up our variables</h2>
We need to declare the variables we'll use to play the music, analyze it and draw a beautiful visualization :)

{% highlight java %}
public static final int WIDTH = 800;
public static final int HEIGHT = 480;

String FILE = &quot;data/justice-new-lands.mp3&quot;;
Mpg123Decoder decoder;
AudioDevice device;
boolean playing = false;

short[] samples = new short[2048];

KissFFT fft;
float[] spectrum = new float[2048];
float[] maxValues = new float[2048];
float[] topValues = new float[2048];

Texture colors;
OrthographicCamera camera;
SpriteBatch batch;

int NB_BARS = 31;
float barWidth = ((float)WIDTH/(float)NB_BARS);
{% endhighlight %}

<em>KissFFT</em> will be used to compute the Fast Fourier Transform from samples we directly read with the decoder. These samples will be streamed by the AudioDevice to make some sounds.
<em>SpriteBatch</em> is used to draw texture on the screen (to draw our spectrum).
<em>maxValues</em> and <em>topValues</em> arrays will be used to make some nice effects ;)
<em>playing</em> boolean will help us to synchronize with the end of the program (dispose).
<em>FILE</em> string corresponds to the audio file we want to analyze. Just put your own music here !
<h2>Initialization</h2>
All of the initialization is made in the create method.We create the instances of the objects previously declared (see source), and define the thread which will be used to play samples and compute the spectrum.

{% highlight java %}
// start a thread for playback&lt;/pre&gt;
Thread playbackThread = new Thread(new Runnable() {
 @Override
 public void run() {
 int readSamples = 0;

 // read until we reach the end of the file
 while(playing &amp;&amp; (readSamples = decoder.readSamples(samples, 0, samples.length)) &gt; 0) {
 // get audio spectrum
 fft.spectrum(samples, spectrum);
 // write the samples to the AudioDevice
 device.writeSamples(samples, 0, readSamples);
 }
 }
 });
 playbackThread.setDaemon(true);
 playbackThread.start();
 playing = true;

{% endhighlight %}

Note that if you want to introduce some filters, a low-pass filter for instance, you can create your own function using the samples array as parameter, work on this array and then place it before the spectrum computation (line 10 on the previous code snippet). To implement a low-pass filter, you can take a look at the Wikipedia page, but there are a lot of resources on the internet.
<h2>Rendering part</h2>
I chose to render the basses in the middle of the screen, made the spectrum symmetric (why not) and added some effects with top and max values, but you can absolutely imagine and draw what you want ! The spectrum array gives you an array of float in the interval [0;255], and you can use these values to do what you want. To draw all the stuff, we'll use a SpriteBatch and theses 3 colors gathered in the following image : Here is the code of the rendering part :

{% highlight java %}

@Override
 public void render() {
 Gdx.gl.glClearColor(0,0,0,1);
 Gdx.gl.glClear(GL10.GL_COLOR_BUFFER_BIT);

 batch.begin();

 for (int i = 0; i &lt; NB_BARS; i++) {
 int histoX = 0;
 if (i&lt;NB_BARS/2) {
   histoX = NB_BARS/2-i;
 } else {
   histoX = i-NB_BARS/2;
 }

 int nb = (samples.length/NB_BARS)/2;
 if (avg(histoX, nb) &gt; maxValues[histoX]) {
   maxValues[histoX] = avg(histoX, nb);
 }

 if (avg(histoX, nb) &gt; topValues[histoX]) {
   topValues[histoX] = avg(histoX, nb);
 }

 batch.draw(colors, i*barWidth, 0, barWidth, scale(avg(histoX, nb)), 0, 0, 16, 5, false, false);
 batch.draw(colors, i*barWidth, scale(maxValues[histoX]), barWidth, 4, 0, 5, 16, 5, false, false);
 batch.draw(colors, i*barWidth, scale(topValues[histoX]), barWidth, 2, 0, 10, 16, 5, false, false);

 topValues[histoX]-=(1.0/3.0);
 }

 batch.end();

 }

{% endhighlight %}

The <em>avg</em> function is used to make the spectrum smoother. It just computes the average of a number of samples. Graphically, we'll draw sort of "stairs". We also scale the height of each bar with a coefficient of 2.0f (yes we can refactor or change it) to use more space. Now you can use your own music file and enjoy the show !
<h2>Conclusion</h2>
Instead of visualizing a spectrum and displaying some representation of it, we can now use these analysis tools to create some filters. For instance, this may be useful for making music-based or rhythm games !
