# CUDA (CuPy Interoperability)

* `pytorch_pfn_extras.cuda.stream(stream)`
    * Context-manager that selects a given stream.
      This context manager also changes the CuPy's default stream if CuPy is available. When CuPy is not available, the functionality is the same as the PyTorch's counterpart, `torch.cuda.stream()`.

* `pytorch_pfn_extras.cuda.use_torch_mempool_in_cupy()`
    * Use PyTorch's memory pool in CuPy.
      If you want to use PyTorch's memory pool and non-default CUDA streams, streams must be created and managed using PyTorch (using `torch.cuda.Stream()` and `pytorch_pfn_extras.cuda.stream(stream)`).
      This feature requires CuPy v8.0+ and PyTorch v1.5+.

* `pytorch_pfn_extras.cuda.use_default_mempool_in_cupy()`
    * Use CuPy's default memory pool in CuPy.

* `pytorch_pfn_extras.from_ndarray(ndarray)`
    * Creates a Tensor from NumPy/CuPy ndarray.

* `pytorch_pfn_extras.as_ndarray(tensor)`
    * Creates a NumPy/CuPy ndarray from Tensor.

* `pytorch_pfn_extras.get_xp(tensor_device_or_ndarray)`
    * Returns ``numpy`` or ``cupy`` module for the given object.

* `pytorch_pfn_extras.as_numpy_dtype(torch_dtype)`
    * Returns NumPy dtype for the given torch dtype.

* `pytorch_pfn_extras.from_numpy_dtype(numpy_dtype)`
    * Returns torch dtype for the given NumPy dtype.
