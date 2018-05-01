.. _pca-xray-calculations:


Calculations
===============================================================================

This is the first part of a lengthy example. In this part, we perform
the calculations using a p38 MAP kinase (MAPK) structural dataset. This will
reproduce the calculations for p38 that were published in [AB09]_.

We will obtain a :class:`.PCA` instance that stores the covariance matrix and
principal modes describing the dominant changes in the dataset. The
:class:`.PCA` instance and principal modes (:class:`.Mode`) can be used as
input for the functions in :mod:`.dynamics` module.


Retrieve dataset
-------------------------------------------------------------------------------

We start by importing everything from the ProDy package:

.. ipython:: python

   from prody import *
   from pylab import *
   ion()

We use a list of PDB identifiers for the structures that we want to
include in our analysis.

.. ipython:: python

   pdbids = ['1A9U', '1BL6', '1BL7', '1BMK', '1DI9', '1IAN', '1KV1', '1KV2',
             '1LEW', '1LEZ', '1M7Q', '1OUK', '1OUY', '1OVE', '1OZ1', '1P38',
             '1R39', '1R3C', '1W7H', '1W82', '1W83', '1W84', '1WBN', '1WBO',
             '1WBS', '1WBT', '1WBV', '1WBW', '1WFC', '1YQJ', '1YW2', '1YWR',
             '1ZYJ', '1ZZ2', '1ZZL', '2BAJ', '2BAK', '2BAL', '2BAQ', '2EWA',
             '2FSL', '2FSM', '2FSO', '2FST', '2GFS', '2GHL', '2GHM', '2GTM',
             '2GTN', '2I0H', '2NPQ', '2OKR', '2OZA', '3HVC', '3MH0', '3MH3',
             '3MH2', '2PUU', '3MGY', '3MH1', '2QD9', '2RG5', '2RG6', '2ZAZ',
             '2ZB0', '2ZB1', '3BV2', '3BV3', '3BX5', '3C5U', '3L8X', '3CTQ',
             '3D7Z', '3D83', '2ONL']


Note that we used a list of identifiers that are different from what was listed
in the supporting material of [AB09]_.  Some structures have been refined and
their identifier have been changed by the Protein Data Bank.
These changes are reflected in the above list.

Also note that it is possible to update this list to include all of the p38
structures currently available in the PDB using the
:func:`.blastPDB` function as follows:

.. ipython:: python

   p38_sequence = '''GLVPRGSHMSQERPTFYRQELNKTIWEVPERYQNLSPVGSGAYGSVCAAFDTKTGHRV
   AVKKLSRPFQSIIHAKRTYRELRLLKHMKHENVIGLLDVFTPARSLEEFNDVYLVTHLMGADLNNIVKCQKLTDDH
   VQFLIYQILRGLKYIHSADIIHRDLKPSNLAVNEDCELKILDFGLARHTDDEMTGYVATRWYRAPEIMLNWMHYNQ
   TVDIWSVGCIMAELLTGRTLFPGTDHIDQLKLILRLVGTPGAELLKKISSESARNYIQSLAQMPKMNFANVFIGAN
   PLAVDLLEKMLVLDSDKRITAAQALAHAYFAQYHDPDDEPVADPYDQSFESRDLLIDEWKSLTYDEVISFVPPPLD
   QEEMES'''


To update list of p38 MAPK PDB files, you can make a blast search as follows:

.. ipython:: python
   :verbatim:

   blast_record = blastPDB(p38_sequence)
   pdbids = blast_record.getHits()

We use the same set of structures to reproduce the results.
After we listed the PDB identifiers, we obtain them using
:func:`.fetchPDB` function as follows:

.. ipython:: python

   pdbfiles = fetchPDB(*pdbids, compressed=False)

``pdbfiles`` variable contains a list of PDB filenames.


Set reference chain
-------------------------------------------------------------------------------

The next step is setting one of the p38 structures as the reference
structure. We use 1p38 chain A. Note that we won't use
all of the resolved residues in this structure. We select only those residues
which are resolved in at least 90% of the dataset.

.. ipython:: python

   ref_structure = parsePDB('1p38')
   ref_selection = ref_structure.select('resnum 5 to 31 36 to 114 122 to '
                                        '169 185 to 351 and calpha')

Retrieve protein chain A from the reference selection:

.. ipython:: python

   ref_chain = ref_selection.getHierView().getChain('A')
   repr(ref_chain)

We use the :func:`.parsePDB` function to parse a PDB file.
This returns a :class:`.AtomGroup` instance. We make a copy
of α-carbon atoms of select residues for analysis.

See :ref:`selections` for making selections.

Prepare ensemble
-------------------------------------------------------------------------------

X-ray structural ensembles are heterogenenous, i.e. different structures
have different sets of unresolved residues. Hence, it is not straightforward
to analyzed them as it would be for NMR models (see :ref:`pca-nmr`).

ProDy has special functions and classes for facilitating efficient analysis
of the PDB X-ray data. In this example we use :func:`.mapOntoChain`
function which returns an :class:`.AtomMap` instance.

See :ref:`atommaps` for more details.

Start a logfile to save screen output:

.. ipython:: python

   startLogfile('p38_pca')

Instantiate an :class:`.PDBEnsemble` object:

.. ipython:: python

   ensemble = PDBEnsemble('p38 X-ray')

Set atoms and reference coordinate set of the ensemble:

.. ipython:: python

   ensemble.setAtoms(ref_chain)
   ensemble.setCoords(ref_chain)

For each PDB file, we find the matching chain and add it to the ensemble:

.. ipython:: python

   for pdbfile in pdbfiles:
       # Parse next PDB file. (only alpha carbons, since it's faster)
       structure = parsePDB(pdbfile, subset='calpha')
       # Get mapping to the reference chain
       mappings = mapOntoChain(structure, ref_chain)
       atommap = mappings[0][0]
       # Add the atommap (mapped coordinates) to the ensemble
       # Note that some structures do not completely map (missing residues)
       # so we pass weights (1 for mapped atoms, 0 for unmapped atoms)
       ensemble.addCoordset(atommap, weights=atommap.getFlags('mapped'))

.. ipython:: python

   repr(ensemble)
   len(ensemble) == len(pdbfiles)

Perform an iterative superimposition:

.. ipython:: python

   ensemble.iterpose()

Close the logfile (file content shows how chains were paired/mapped):

.. ipython:: python

   closeLogfile('p38_pca')

Save coordinates
-------------------------------------------------------------------------------

We use :class:`.PDBEnsemble` to store coordinates of the X-ray
structures. The :class:`.PDBEnsemble` instances do not store any
other atomic data. If we want to write aligned coordinates into a file, we
need to pass the coordinates to an :class:`.AtomGroup` instance.
Then we use :func:`.writePDB` function to save coordinates:

.. ipython:: python

   writePDB('p38_xray_ensemble.pdb', ensemble)


PCA calculations
-------------------------------------------------------------------------------

Once the coordinate data are prepared, it is straightforward to perform the
:class:`.PCA` calculations:

.. ipython:: python

   pca = PCA('p38 xray')           # Instantiate a PCA instance
   pca.buildCovariance(ensemble)   # Build covariance for the ensemble
   pca.calcModes()                 # Calculate modes (20 of the by default)

**Approximate method**

In the following we are using singular value decomposition for faster
and more memory efficient calculation of principal modes:

.. ipython:: python

   pca_svd = PCA('p38 svd')
   pca_svd.performSVD(ensemble)

The resulting eigenvalues and eigenvectors may show small differences due to
missing atoms in the datasets:

.. ipython:: python

   abs(pca_svd.getEigvals()[:20] - pca.getEigvals()).max()
   abs(calcOverlap(pca, pca_svd).diagonal()[:20]).min()

Note that building and diagonalizing the covariance matrix is the preferred
method for heterogeneous ensembles. For NMR models or MD trajectories SVD
method may be preferred over covariance method.

ANM calculations
-------------------------------------------------------------------------------

To perform :class:`.ANM` calculations:

.. ipython:: python

   anm = ANM('1p38')             # Instantiate a ANM instance
   anm.buildHessian(ref_chain)   # Build Hessian for the reference chain
   anm.calcModes()               # Calculate slowest non-trivial 20 modes

Save your work
-------------------------------------------------------------------------------

Calculated data can be saved in a ProDy internal format
to use in a later session or to share it with others.

If you are in an interactive Python session, and wish to continue without
leaving your session, you do not need to save the data. Saving data is useful
if you want to use it in another session or at a later time, or if you want
to share it with others.

.. ipython:: python

   saveModel(pca)
   saveModel(anm)
   saveEnsemble(ensemble)
   writePDB('p38_ref_chain.pdb', ref_chain)

We use the :func:`.saveModel` and :func:`.saveEnsemble` functions to save
calculated data. In :ref:`pca-xray-analysis`, we will use the
:func:`.loadModel` and :func:`.loadEnsemble` functions to load the data.
