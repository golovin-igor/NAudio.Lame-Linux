# NAudio.Lame
[![Build Status](https://github.com/Corey-M/NAudio.Lame/workflows/.NET%20Core%202.2/badge.svg)](https://github.com/Corey-M/NAudio.Lame/actions?workflow=.NET%20Core%202.2)
## Description

This fork was made to support .net core wrapper for libmp3lame on Linux (x64).
Code was tested on Ubuntu 19 x64, but in general this should work on most platforms

Wrapper for `libmp3lame` to add MP3 encoding support to NAudio on Linux.

**IMPORTANT:** Currently it is not cross-compatible, for Windows please use the original.
It is not (yet) compatible with original source from https://github.com/Corey-M/NAudio.Lame/
It was not tested on x86 platform

## Usage

Steps to run on Linux (tested on Ubuntu 19 with .net core 2.2):

1. sudo apt install lame
2. sudo apt install libmp3lame-dev

3. dotnet yourApp.dll


### Sample Code

Here is a very simple codec class to convert a WAV file to and from MP3:

    using System.IO;
    using NAudio.Wave;
    using NAudio.Lame;

    public static class Codec
    {
        // Convert WAV to MP3 using libmp3lame library
        public static void WaveToMP3(string waveFileName, string mp3FileName, int bitRate = 128)
        {
            using (var reader = new AudioFileReader(waveFileName))
            using (var writer = new LameMP3FileWriter(mp3FileName, reader.WaveFormat, bitRate))
                reader.CopyTo(writer);
        }

        // Convert MP3 file to WAV using NAudio classes only
        public static void MP3ToWave(string mp3FileName, string waveFileName)
        {
            using (var reader = new Mp3FileReader(mp3FileName))
            using (var writer = new WaveFileWriter(waveFileName, reader.WaveFormat))
                reader.CopyTo(writer);
        }
    }


### Sample Code

    using NAudio.Wave;
    using NAudio.Lame;
    using System;

    class Program
    {
        static void Main(string[] args)
        {
            ID3TagData tag = new ID3TagData 
            {
                Title = "A Test File",
                Artist = "Microsoft",
                Album = "Windows 7",
                Year = "2009",
                Comment = "Test only.",
                Genre = LameMP3FileWriter.Genres[1],
                Subtitle = "From the Calligraphy theme"
            };

            using (var reader = new AudioFileReader(@"test.wav"))
            using (var writer = new LameMP3FileWriter(@"test.mp3", reader.WaveFormat, 128, tag))
            {
                reader.CopyTo(writer);
            }
        }
    }

### Sample Code

    using NAudio.Wave;
    using NAudio.Lame;
    using System;

    class Program
    {
        // For calculation of progress percentage, total bytes to be input
        static long input_length = 0;

        static void Main(string[] args)
        {
            using (var reader = new NAudio.Wave.AudioFileReader(@"/var/tmp/testwave.wav"))
            using (var writer = new NAudio.Lame.LameMP3FileWriter(@"/var/tmp/encoded.mp3", reader.WaveFormat, NAudio.Lame.LAMEPreset.V3))
            {
                writer.MinProgressTime = 250;
                input_length = reader.Length;
                writer.OnProgress += writer_OnProgress;
                reader.CopyTo(writer);
            }
        }

        static void writer_OnProgress(object writer, long inputBytes, long outputBytes, bool finished)
        {
            string msg = string.Format("Progress: {0:0.0}%, Output: {1:#,0} bytes, Ratio: 1:{2:0.0}",
                (inputBytes * 100.0) / input_length,
                outputBytes,
                ((double)inputBytes) / Math.Max(1, outputBytes));

            Console.Write("\r{0," + (Console.BufferWidth - 1).ToString() + "}\r{1}", "", msg);
            if (finished)
                Console.WriteLine();
        }
    }


