.. |k1| replace:: ``k``\ :sub:`1`
.. |k2| replace:: ``k``\ :sub:`2`
.. |k3| replace:: ``k``\ :sub:`3`
.. |nq| replace:: ``n``\ :sub:`q`

The BM25 Weighting Scheme
=========================

This is a technical note about the BM25 weighting scheme, which is the
default weighting scheme used by Xapian. Recent TREC tests have shown
BM25 to be the best of the known probabilistic weighting schemes. In
case you're wondering, the BM simply stands for "Best Match".

We'll follow the evolution from the "traditional" probabilistic
weighting scheme (as described in the `1976 Robertson/Sparck Jones
paper <http://www.soi.city.ac.uk/~ser/papers/RSJ76.pdf>`_) through to
BM25.

The Traditional Probabilistic Weighting Scheme
----------------------------------------------

In its most general form, the traditional probabilistic term weighting
function is:

.. raw:: html

   <table border=0><tr valign=center>
   <td>
   <tt><center>
   <u>(k<sub>3</sub>+1)q</u><br>(k<sub>3</sub>+q)</center></tt>
   </td>
   <td><tt>&middot;</tt></td>
   <td>
     <tt><center>
   <u>(k<sub>1</sub>+1)f</u><br>(k<sub>1</sub>L+f)</center></tt>
   </td>
   <td><tt>&middot;log</tt></td>
   <td><tt><center>
   <u>(r+0.5)(N-n-R+r+0.5)</u><br>(n-r+0.5)(R-r+0.5)</center></tt>
   </td>
   <td>
   <tt>
   &nbsp;...(1)
   </tt>
   </td>
   </tr></table>

where

  * |k1|, |k3| are constants,
  * q is the wqf, the within query frequency,
  * f is the wdf, the within document frequency,
  * n is the number of documents in the collection indexed by this term,
  * N is the total number of documents in the collection,
  * r is the number of relevant documents indexed by this term,
  * R is the total number of relevant documents,
  * L is the normalised document length (i.e. the length of this document
    divided by the average length of documents in the collection).

The factors (|k3|\ ``+1``) and (|k1|\ ``+1``) are unnecessary here, but help
scale the weights, so the first component is 1 when ``q=1`` etc. But
they are critical below when we add an extra item to the sum of term
weights.

BM11
----

Stephen Robertson's BM11 uses formula ``(1)`` for the term weights, but
adds an extra item to the sum of term weights to give the overall
document score:

.. raw:: html

    <table border=0><tr valign=center> <td><tt>k<sub>2</sub>
    n<sub>q</sub></tt></td> <td> <tt><center>
    <u>(1-L)</u><br>(1+L)</center></tt> </td> <td> <tt> &nbsp;...(2) </tt> </td>
    </tr></table>

where:

- |nq| is the number of terms in the query (the query length),
- |k2| is yet another constant.

Note that this extra item is zero when ``L=1``.

BM15
----

BM15 is BM11 with the |k1|\ ``+f`` in place of |k1|\ ``L+f`` in ``(1)``.

BM25
----

BM25 combines the B11 and B15 with a scaling factor, b, which turns BM15
into BM11 as it moves from 0 to 1:

.. raw:: html

    <table border=0><tr valign=center> <td> <tt><center>
    <u>(k<sub>3</sub>+1)q</u><br>(k<sub>3</sub>+q)</center></tt> </td>
    <td><tt>&middot;</tt></td> <td> <tt><center>
    <u>(k<sub>1</sub>+1)f</u><br>(K+f)</center></tt> </td>
    <td><tt>&middot;log</tt></td> <td><tt><center>
    <u>(r+0.5)(N-n-R+r+0.5)</u><br>(n-r+0.5)(R-r+0.5)</center></tt> </td> <td>
    <tt> &nbsp;...(3) </tt> </td> </tr></table>

where:

``K=``\ |k1|\ ``(bL+(1-b))``

BM25 originally introduced another constant, as a power to which f and K
are raised. However, Stephen remarks that powers other than 1 were *'not
helpful'*, and other tests confirm this, so Xapian's implementation of
BM25 ignores this.

``(2)`` and ``(3)`` make up BM25, with which Stephen has had so much
recent success.

This does all seem somewhat ad-hoc, with so many unknown constants in
the formula. But note that with |k2|\ ``=0`` and ``b=1`` we get the
traditional formula anyway.

The default parameter values Xapian uses are |k1|\ ``=1``, |k2|\ ``=0``,
|k3|\ ``=1``, and ``b=0.5``. These are reasonable defaults, but the
optimum values will vary with both the documents being searched and the
type of queries, so you may be able to improve the effectiveness of your
search system by tuning the values of these parameters.

In Xapian, we also apply a floor to L (0.5 by default) which helps stop
tiny documents get ridiculously high weights. And the matcher wants the
extra item in the sum to be positive, so we add |k2|\ |nq| (constant for a
given query) to ``(2)`` to give:

.. raw:: html

    <table border=0><tr valign=center> <td> <tt><center> <u>2 k<sub>2</sub>
    n<sub>q</sub></u><br>(1 + L)</center></tt> </td> <td> <tt> &nbsp;...(4)
    </tt> </td> </tr></table>