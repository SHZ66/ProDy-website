.. _pfamaccess:

Pfam Access
===============================================================================

The part shows how to access Pfam database. You can search protein family
accession numbers and information using a sequence or PDB/UniProt identifiers.
MSA files for families of interest can be retrieved in a number of formats.

.. ipython:: python

   from prody import *
   from matplotlib.pylab import *
   ion()  # turn interactive mode on

Search Pfam
-------------------------------------------------------------------------------

We use :func:`.searchPfam` for searching.  Valid inputs are UniProt ID,
e.g. :uniprot:`PIWI_ARCFU`, or PDB identifier, e.g. :pdb:`3luc` or ``"3lucA"``
with chain identifier.

Matching Pfam accession (one or more) as keys will map to a dictionary that
contains locations (alignment start, end, evalue etc), pfam family type,
accession and id.


We query Pfam using the :func:`.searchPfam`. with a UniProt ID.

.. ipython::

   matches = searchPfam('PIWI_ARCFU')


It is a good practice to save this record on disk, as NCBI may not respond to
repeated searches for the same sequence. We can do this using Python standard
library :mod:`pickle` as follows:

.. ipython:: python

   import pickle

Record is save using :func:`~pickle.dump` function into an open file:

.. ipython::

   pickle.dump(matches, open('pfam_search_PIWI_ARCFU.pkl', 'w'))


Then, it can be loaded using :func:`~pickle.load` function:

.. ipython:: python

   matches = pickle.load(open('pfam_search_PIWI_ARCFU.pkl'))
   matches


.. Input can also be a protein sequence or a file containing the sequence:

.. .. ipython::
   :verbatim:

..   In [1]: sequence = ('PMFIVNTNVPRASVPDGFLSELTQQLAQATGKPPQYIAVHVVPDQLMAFGGSSE'
      ...: 'PCALCSLHSIGKIGGAQNRSYSKLLCGLLAERLRISPDRVYINYYDMNAANVGWNNSTFA')


..   In [2]: matches = searchPfam(sequence)


..   In [10]: pickle.dump(matches, open('pfam_search_sequence.pkl', 'w'))

.. .. ipython:: python

..   matches = pickle.load(open('pfam_search_sequence.pkl'))
   matches

.. Input sequence cannot have gaps and should be at least 16 characters long.

.. For sequence searches, we can pass additional parameters to :func:`.searchPfam`
like *search_b* which will search pfam B and *skip_a* that will not search
pfamA database. Additional parameters include *ga* that uses gathering
threshold instead of e-value, *evalue* cutoff can also be specified and
*timeout* that can be set higher especially when searching larger
sequences, default is ``timeout=60`` seconds.

.. .. ipython::
   :verbatim:

..   In [1]: matches = searchPfam(sequence, search_b=True, evalue=2.0)


Retrieve MSA files
-------------------------------------------------------------------------------

Data from Pfam database can be fetched using :func:`.fetchPfamMSA`.

Valid inputs are Pfam ID, e.g. :pfam:`Piwi`, or Pfam accession, e.g.
:pfam:`PF02171` obtained from :func:`.searchPfam`.

Alignment type can be ``"full'`` (default), ``"seed"``, ``"ncbi"`` or
``"metagenomics"`` or ``"rp15"`` or ``"rp35"`` or ``"rp55"`` or ``"rp75"``.

.. ipython::

   fetchPfamMSA('piwi', alignment='seed')

A compressed file can be downloaded by setting ``compressed=True``.
The ``format`` of the MSA can be of ``"selex"`` (default), ``"stockholm"`` or
``"fasta"``.  This will return the path of the downloaded MSA file.
The ``output`` name can be specified, for by default it will have
``"accession/ID_alignment.format"``.

Note that in this case we passed a folder name, the downloaded file is saved
in this folder, after it is created if it did not exist. Also longer timeouts
are necessary for larger families. Some other parameters like ``gap``,
``order`` or ``inserts`` can be set, as shown in the following example.

.. ipython::

   fetchPfamMSA('PF02171', compressed=True, gaps='mixed',
      ...: inserts='lower', order='alphabetical', format='fasta')