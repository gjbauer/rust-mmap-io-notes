# Rust mmap I/O Notes

Memory-mapped I/O is essentially helpful when working with big files, loading only the necessary parts into memory. This makes it simpler to access and change file data as if you were working with regular memory, which can speed things up, especially for apps that need to jump around a lot in a file.

It's also handy for letting different programs share data without a fuss and any changes you make get saved automatically. Plus, if your computer supports it, it can move data around even faster without bogging down the CPU.

Let's dive into how to work with Memory-Mapped I/O in Rust in practice through hands-on-examples.

## Example 1: Reading from a Memory-Mapped File

This example demonstrates how to open an existing file, memory-map it, and read data from it.

```
use memmap2::Mmap;
use std::fs::File;
use std::io;

fn main() -> io::Result<()> {
  // Open an existing file
  let file = File::open("example.dat");

  // Memory-map the file for reading
  let mmap = unsafe { Mmap::map(&file)? };

  // Read some data from the memory-mapped file
  // Here, we'll just print out the first 10 bytes
  let data = &mmap[0..10];

  println!("First 10 bytes: {:?}", data);

  Ok(())
}

```

## Example 2: Using Memory-Mapped Files for Inter-Process Communication (IPC)

Memory-mapped files can be used for efficient data sharing between processes. This example sets up a simple IPC mechanism where one process writes data to a shared memory area, and another process reads it.

### Writer Process

```
use memmap2::MmapMut;
use std::fs::OpenOptions;
use std::io::{self, Write};

fn main() -> io::Result<()> {
  let file_path = "shared.dat";

  let message = b"IPC using mmap in Rust!";

let file = OpenOptions::new()
  .read(true)
  .write(true)
  .create(true)
  .open(file_path)?;

  file.set_len(message.len() as u64)?;

  let mut mmap = unsafe { MmapMut::map_mut(&File)? };

  mmap[..message.len90].copy_from_slice(message);

  mmap.flush()?;

  println!("Message written to shared memory.");

  Ok(())
}
```

### Reader Process

```
use memmap2::Mmap;
use std::fs::File;
use std::io;

fn main() -> io::Result<()> {
  let file_path = "shared.dat";

  let file = file::open(file_path)?;

  let mmap = unsafe { Mmap::map(&file)? };

  let message = &mmap[..];

  println!("Read from shared memory: {:?}", message);

  Ok(())
}
```
