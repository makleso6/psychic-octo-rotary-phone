Устройство: cuda
Train: 102438 чанков | Val: 11382 чанков
Generator params: 44,236,305
Discriminator params: 1,170,435
Traceback (most recent call last):                                                                                            
  File "/home/makleso6/fictional-garbanzo/train_gan.py", line 204, in <module>
    main()
  File "/home/makleso6/fictional-garbanzo/train_gan.py", line 150, in main
    scaler_g.scale(loss_g).backward()
  File "/home/makleso6/fictional-garbanzo/venv/lib/python3.11/site-packages/torch/_tensor.py", line 631, in backward
    torch.autograd.backward(
  File "/home/makleso6/fictional-garbanzo/venv/lib/python3.11/site-packages/torch/autograd/__init__.py", line 381, in backward
    _engine_run_backward(
  File "/home/makleso6/fictional-garbanzo/venv/lib/python3.11/site-packages/torch/autograd/graph.py", line 869, in _engine_run_backward
    return Variable._execution_engine.run_backward(  # Calls into the C++ engine to run the backward pass
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
torch.OutOfMemoryError: CUDA out of memory. Tried to allocate 520.00 MiB. GPU 0 has a total capacity of 15.47 GiB of which 191.62 MiB is free. Including non-PyTorch memory, this process has 14.68 GiB memory in use. Of the allocated memory 12.12 GiB is allocated by PyTorch, and 2.22 GiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://docs.pytorch.org/docs/stable/notes/cuda.html#optimizing-memory-usage-with-pytorch-cuda-alloc-conf)
