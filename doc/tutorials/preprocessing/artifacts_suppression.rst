Artifact suppression
====================
Typical M/EEG data contain some artifacts; we first identify them, then we remove them.
Typical artifacts includes:
- Blinks (recorded through EOG)
- Heartbeats (recorded through ECG)

To extract the spatio-temporal contribution of blinks and heartbeats, you
can use either ICA or SSP; we recommend to use ICA.

Independent Component Analysis (ICA)
------------------------------------
ICA finds directions in the feature space
corresponding to projections with high non-Gaussianity. We thus obtain
a decomposition into independent components, and the artifacts contribution is
localized in only a small number of components.
These components have to be correctly identified and removed.

If EOG or ECG recordings are available, they can be used in ICA to automatically
select the corresponding artifact components from the decomposition. To do so,
you have to first build an Epoch object around blink or heartbeat events, and then
compute a score that quantify how much the Epoch is correlated to each independent
component.

    >>> ecg_epochs = mne.preprocessing.create_ecg_epochs(raw)
    >>> # initialize the ICA estimator and fit it on the data
    >>> ica = mne.preprocessing.ICA(n_components=0.95, method='fastica')
    >>> ica.fit(raw, picks=picks)
    >>> ecg_inds, scores = ica.find_bads_ecg(ecg_epochs, method='ctps')
    >>> # we remove only 3 components
    >>> ica.exclude += ecg_inds[:3]

.. figure:: _images/sphx_glr_plot_ica_from_raw_005.png
   :target: auto_tutorials/plot_ica_from_raw.html
   :scale: 50%

Examples:
    ../../auto_tutorials/plot_ica_from_raw.html
    auto_examples/preprocessing/plot_find_ecg_artifacts.html
    auto_examples/preprocessing/plot_find_eog_artifacts.html
    auto_examples/preprocessing/plot_eog_artifact_histogram.html

If EOG or ECG recordings are not available, you can visually select the artifact
components. This can be done on the independent components time series, looking for
the characteristic shapes of blinks and heartbeats artifacts, or on the independent
components spatial distribution.
You can use the manual identification on one subject in order to automatically
find corresponding components in other subject, using
:func:`CORRMAP <mne.preprocessing.ica.corrmap>`.

Examples:
    auto_examples/preprocessing/plot_corrmap_detection.html
    auto_examples/preprocessing/plot_run_ica.html

ICA-based artifact rejection is done using the :class:`mne.preprocessing.ICA`
class, see the :ref:`manual/preprocessing/ica` section in the manual for more information.


Signal-Space Projection (SSP)
-----------------------------

Instead of using ICA, you can also use Signal-Space Projection (SSP) to extract artifacts.
SSP-based rejection is done using the
:func:`compute_proj_ecg <mne.preprocessing.compute_proj_ecg>` and
:func:`compute_proj_eog <mne.preprocessing.compute_proj_eog>` methods,
see :ref:`manual/preprocessing/ssp` section in the manual for more information.


    >>> ecg_proj, ecg_event = mne.preprocessing.compute_proj_ecg(raw)
