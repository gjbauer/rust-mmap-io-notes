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
