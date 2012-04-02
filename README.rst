==============================================================
PyZerem -- Dataflow programming in Python with metaprogramming
==============================================================

This is, for now, just a placeholder. I'm putting my ideas here,
in hope that someone with time and interest -- hopefully a future
me -- will pick them up and run with them. At this point, there's
no actual code here. Just thoughts.

Concepts
========

A Dataflow specification is made of slots where data can
come in, and processes that are triggered in response to
such data becoming available.

Basic idea
----------

::

  from zerem import *

  # Classes deriving from zerem.Flow are special
  class MyFlow(Flow):

    # A Flow class can have slots
    input1 = Slot(...)
    input2 = Slot(...)
    intermediary = Slot(...)
    output = Slot(...)

    # Functions marked as processes are invoked automatically
    # when their inputs become available. 
    # Inputs to processes are identified by arguments with 
    # the names of slots. 

    @process
    def step1(self, input1):

      # Besides invocation, this is a regular function. the
      # argument it receives is any python object. Let's assume
      # it is a string
      x = input1.tolower()
      if x.startswith('a'):
         y = 'foo'
      else:
         y = 'bar'

      # Processes don't return values; they can push values into
      # slots, though. Assignment to a slot pushes a value into it.

      self.intermediary = y

    @process
    def step2(self, input2, intermediary):
    
      # This will be called automatically when both input2 and 
      # intermediary are available (intermediary will become
      # available after step1 above completes)

      self.output = input2 + intermediary

More general details
--------------------

Slots can have different semantics. The default should be a
queue, where when a value is consumed by a process it is
popped off. However, there can also be a "keep the last assigned
value always available" (change should still trigger processes).

Processes are triggered when one argument changes and all arguments
are available. Non-mandatory arguments can be specified by supplying
defaults::

  @process
  def step(self, a=None, b=None):
    pass

``step`` will be triggered when either of its arguments become available.

Processes should have no arguments which aren't slots.

Lifecycle
---------

I'm not clear yet on how instances of flows work. I'm thinking
of initiator processes, but that requires to have slots
on the flow class itself rather than the instance.

Another issue is the ``output`` slot defined earlier. Who consumes
it? how? Not clear yet. There should probably be a way to connect
it to somebody's input slot, and that way should be available out
of the flow definitions, so they can be mixed-and-matched.

Previous work
=============

This came about, mainly, in response to Pythonect_; it made me
think, and my thoughts were inspired by (an old version of)
the Tersus_ dataflow engine on one hand, and `Django models`_
on the other.

Also of note are other tools for dataflow programming in Python:
PyF_, Trellis_ and Pypes_ seem to have a lot of semantics covered, and
py-dataflow_ seems to have arrived at some similar metaprogramming
concepts.

.. _Pythonect: http://www.pythonect.org/
.. _Tersus: http://www.tersus.com/
.. _`Django models`: https://docs.djangoproject.com/en/dev/topics/db/models/
.. _PyF: http://pyfproject.org/
.. _Pypes: https://github.com/diji/pypes/
.. _Trellis: http://peak.telecommunity.com/DevCenter/Trellis
.. _py-dataflow: https://github.com/gfxmonk/py-dataflow

