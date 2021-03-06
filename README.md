# Sreview

This is sreview, a video review system. It takes input files, stores
their lengths in a database, combines those lengths and their starttime
with a schedule it has of an event to see which talks are fully
recorded, and creates a preview. After that, magic happens, and
eventually a fully transcoded quality video rolls out of the system.

Note that sreview is currently in beta state. I haven't tested
everything yet, but hopefully I'll be using it fairly soon in
production.

## States

Sreview is fairly minimalistic; it tries to assume as little as possible
about video workflow. There is, however, a state machine that you should
be aware of:

    files_missing
    partial_files_found
    files_found
    cut_pending
    cut_ready
    generating_previews
    preview
    review_done
    generating_data
    waiting
    uploading
    done
    broken

Every talk is in one of the above states. The following is a list of
possible states, with their meaning:

- `files_missing`: no files have been found as of yet. Talks should
  initially be in this state.
- `partial_files_found`: some files have been found, but not all of
  them. This may be because some data was lost, or because the talk is
  not finished yet.
- `cut_pending`: the `cut_talk` script is running, or scheduled to
  start.
- `cut_ready`: the `cut_talk` script has finished.
- `generating_previews`: the `previews` script is running, or scheduled
  to start.
- `preview`: the `previews` script has finished. This talk is now ready
  for review by a human being (the webinterface is necessary for this
  step).
- `review_done`: human review has finished
- `generating_data`: the `transcode` script is running, or scheduled to
  start.
- `waiting`: the `transcode` script has finished
- `uploading`: the files are being uploaded.
- `done`: the talk has been fully completed, all files should be
  published
- `broken`: sreview will not automatically switch a talk to this state,
  but it can be used to mark talks that are lost forever and should not
  be considered anymore.

## Components

Sreview consists of two major components: a webinterface (written in
Perl with Mojolicious), and a backend which consists of another set of
perl scripts.

To run the webinterface in a test environment, copy the
`config.pl.template` file in the web directory to `config.pl`, edit it,
run "./sreview daemon", and browse to the URL given. To run the
webinterface in production, see
[Mojo::Server::Hypnotoad](http://mojolicious.org/perldoc/Mojo/Server/Hypnotoad)
(or some of the other guides over there).

To run the backend, it is recommended that you install gridengine first.
In theory, the backend *should* work without gridengine, but that is not
as well tested. Additionally, you will then need to run a dispatcher per
CPU, rather than having just one dispatcher in the whole network.

Once gridengine has been installed, copy the `config.pl.template` file
in the scripts directory to `config.pl`, edit it, and run `perl
dispatch`.

If you want to modify the output formats, you should edit the
`transcode` script where you can add and/or remove gstreamer pipelines.

If you want to modify the look and feel of the webinterface, you should
edit the files in the web/templates directory.

If you have any issues with sreview, please file an issue (or better
yet, a pull request) on the github issue tracker.
