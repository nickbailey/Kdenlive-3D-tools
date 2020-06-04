# Kdenlive-3D-tools

## What's this?

Some consumer camcorders come with the lenses or adaptors to permit 3D stereography.
The result is stored as an MPEG transport stream ("MTS" file) and that makes it hard
to edit with non-proprietary video editors such as [Kdenlive](https://kdenlive.org).

Such files store two video streams. The first is a conventional AVC stream
which records the left eye. The right eye information from my camcorder is recorded as an
[AVCHD](https://en.wikipedia.org/wiki/AVCHD) dependent stream. My understanding is this
contains only differences between the right and left eye vide; there are no i-frames.

This structure retains compatibility with existing video applications because they
only recognise the left-eye information and treat the file as a regular 2D one.

3D is, at best, niche. It is therefore unlikely there will be any core work in Kdenlive
to support 3D files presented in this format. However, stereo videography is still
being done with two cameras, and it would be nice to edit the results Kdenlive.

The plan is to enable Kdenilve to be able to produce a final 3D cut using the following
workflow:

1. Extract the video and audio streams from the AVCHD container.
1. Create a right-eye standalone video stream for use later.
1. Edit the MTS sources using the left-eye stream as normal.
1. Export a script which does the editing.
1. Run the tools here to perform the render. This will invoke the following processes:
    1. Run kdenlive's script against the source files producing a left-eye output file.
    1. Adjust the environment so that the edit operation
       will operate on the right-eye video files.
    1. Combine the output from the left and right eye sources to a form
       displayable on a 3D TV or projector (not necessarily AVCHD; possibly
       side-by-side or top-and-bottom frames)
1. declare success.

It is recognised that this is not an efficient way of doing things, but it should at least
give back some creative control of 3D content to those who are no longer able to run
the supplied "editing suites" that came with the original cameras.

## Project status

Just starting up and targetting Linux.

I've managed to extract the right eye as an MPV which plays stand-alone,
but not using a fully open-source tool chain. There's a windows executable I'm using to decode the dependent
stream in to a stand-alone file once the streams have been extracated, which runs in
[Wine](https://wiki.winehq.org/).

