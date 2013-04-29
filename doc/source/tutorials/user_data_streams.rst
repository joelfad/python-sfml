User data streams
=================

.. contents:: :local:

Introduction
------------
SFML has several resource classes: images, fonts, sounds, etc. In most
programs, these resources will be loaded from files, with the help of their
`from_file` function. In a few other situations, resources will be packed
directly into the script or in a big data file, and loaded from memory with
`from_memory`. These functions cover almost all the possible use cases -- but
not all.

Sometimes you want to load files from unusual places, such as a
compressed/encrypted archive, or a remote network place for example. For these
special situations, SFML provides a third loading function: `from_stream`. This
function reads data from an abstract :class:`.InputStream` class, which allows
you to define your own implementations.

In this tutorial you'll learn how to write and use your own derived input
stream.

And standard streams?
---------------------
Like many other languages, Python already has classes for input data streams,
the `io` module defines them. In fact, :class`InputStream` is a derivated
standard stream.

Therefore, you can use any of them, they're both compatible. If standard stream
classes are not very user friendly to you, use :class:`.InputStream`, it will
also ensure a common interface across Python version

That's why SFML provides its own stream interface, which is hopefully a lot
more simple and fast.

InputStream
-----------
The :class:`InputStream` class declares four virtual functions: ::

   class InputStream:

      read(self, length): pass
      seek(self, position): pass
      tell(self): pass
      __len__(self): pass

**read** must extract size bytes of data from the stream, and return them
-1 on error.

**seek** must change the current reading position in the stream; its position
argument is the absolute byte offset to jump to (so it is relative to the
beginning of the data, not to the current position); it returns the new
position, or -1 on error.

**tell** must return the current reading position (in bytes) in the stream,
or -1 on error.

**__len__** must return the total size (in bytes) of the data which is
contained in the stream, or -1 on error.

To create your own working stream, you must implement all these four functions
according to their requirements.

An example
----------
Here is a complete and working implementation of a custom input stream. It's
not very useful: we'll write a stream that reads data from a file, `FileStream`.
But it is simple enough so that you can focus on how the code works, and not
get lost in implementation details.

In this example we'll use the standard file API, so we have a member containing
the file object. We also add a default constructor, a destructor, and a
function to open the file.

Here is the implementation: ::

   import io
   import sfml.system as sf

   class FileStream(sf.InputStream):
      def __init__(self):
         sf.InputStream.__init__(self)

         self._file = None

      def __len__(self):
         cur = self.tell()
         self._file.seek(0, io.SEEK_END)
         length = self.tell()
         self.seek(cur)

      def open(self, filename):
         self._file = open(filename, "rb")

      def read(self, length):
         return self._file.read(length)

      def seek(self, position):
         self._file.seek(position)
         return self.tell()

      def tell(self):
         length = self._file.tell()

Note that, as explained above, all functions return -1 on error.

Don't forget to check the forum and wiki, chances are that another user already
wrote a :class:`.InputStream` class that matches your needs. And if you write a
new one and feel like it could be useful outside your project, don't hesitate
to share!

Using your stream
-----------------
Using a custom stream class is straight-forward: instanciate it, and pass it to
the `from_stream` function of the object that you want to load. ::

   stream = FileStream()
   stream.open("image.png")

   texture = sf.Texture.from_stream(stream)

Common mistakes
---------------
Some resource classes are not loaded completely after `from_stream` has been
called. Instead, they continue to read from their data source as long as they
are used. This is the case for :class:`.Music`, which streams audio samples as
they are played, and for :class:`.Font`, which loads glyphs on the fly depending
on the texts content.

As a consequence, the stream instance that you used to load a music or a font,
as well as its data source, must remain alive as long as the resource uses it.
If it is destroyed while still being used, the behaviour is undefined (can be a
crash, corrupt data, or nothing visible).

Another common mistake is to return whatever the internal functions return
directly, but sometimes it doesn't match what SFML expects. For example, in the
`FileStream` example above, one might be tempted to write the seek function as
follows: ::

   def seek(self, position):
      return self._file.seek(position, io.SEEK_SET)
