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

  mmap[..message.len()].copy_from_slice(message);

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

## Example 3: Modifying a Memory-Mapped File

This example shows how to modify part of a memory-mapped file, demonstrating an in-place update.

```
use memmap2::MmapMut;
use std::fs::OpenOptions;
use std::io;

fn main() -> io::Result<()> {
  let file_path = "example.dat";

  let file = OpenOptions::new()
    .read(true)
    .write(true)
    .open(file_path)?;
  let mut mmap = unsafe { MmapMut::map_mut(&file)? };

  // Modify a portion of the mapped memory
  //let new_data = b"Rust";

  for (i, byte) in new_data.iter().enumerate() {
    mmap[i] = *byte;
  }

  mmap.flush()?;

  println!("Memory-mapped file updated.");

  Ok(())
}

```

### Notes
  * The `unsafe` block is necessary for memory-mapped operations because they involve direct manipulation of memory, which can lead to undefined behavior if not used carefully.
  * Always ensure that the file size is adequate for the operations you intend to perform. Trying to access memory outside the bounds of the mapped file can cause your program to crash.
  * Remember to handle errors and edge cases in a real-world application, such as checking if the file exists before attempting to read from it or ensuring that file modifications do not exceed the file size.

## Example 4: Concurrently Reading from a Memory-Mapped File

Memory-mapped files can be efficiently used by multiple threads to read data concurrently due to the way the operating system handles page caching. Here's an example that demonstrates concurrent reads:

```
use memmap2::Mmap;
use std::fs::File;
use std::io;
use std::sync::Arc;
use std::thread;

fn main() -> io::Result<()> {
  let file = File::open("example.dat")?;

  let mmap = unsafe { Mmap::map(&file)? };

  let mmap_arc = Arc::new(mmap);

  let mut handles = vec![];

  for _ in 0..4 {
    let mmap_clone = Arc::clone(&mmap_arc);
    let handle = thread::spawn(move || {
      // Each thread reads from the memory-mapped file
      let data = &mmap_clone[0..10];  // Example: Read first 10 bytes
      println!("Thread read: {:?}", data);
    });

    handles.push(handle);
  }

  for handle in handles {
    handle.join().unwrap();
  }

  Ok(())
}
```

## Example 5: Using Memory-Mapped Files for Efficient Large Data Manipulation

Memory-mapped files are particularly useful for working with large data sets, as they allow for random access without loading the entire file into memory. Here's an example that manipulates a large file:

```
use memmap2::MmapMut;
use std::fs::OpenOptions;
use std::io;

fn manipulate_large_file(file_path: &str) -> io::Result<()> {
  let file = OpenOptions::new()
    .read(true)
    .write(true)
    .open(file_path)?;

  let mut mmap = unsafe { MmapMut::map_mut(&file)? };

  // Example manipulation: zero out every other byte in a large file
  for i in (0..map.len()).step_by(2) {
    mmap[i] = 0;
  }

  mmap.flush()?; // Ensure changes are written back to the file

  Ok(())
}

fn main() -> io::Result<()> {
  let file_path = "large_sample.dat";
  manipulate_large_file(file_path);
}
```

## Example 6: Memory-Mapped Files for Ring Buffers

Memory-mapped files can be used to implement a ring buffer (circular buffer), which is useful for scenarios where a fixed-size buffer is continuously written to and read from, such as logging systems or stream processing.

```
use memmap2::MmapMut;
use std::fs::{File, OpenOptions};
use std::io;

struct RingBuffer {
  mmap: MmapMut,
  capacity: usize,
  head: usize,
  tail: usize,
}

impl RingBuffer {
  fn new(file_path: &str, size: usize) -> io::Result<Self> {
    let file = OpenOptions::new()
      .read(true)
      .write(true)
      .create(true)
      .open(file_path)?;

    file.set_len(size as u64)?;

    let mmap = unsafe { MmapMut::map_mut(&file)? };

    Od(Self {
      mmap,
      capacity: size,
      head: 0,
      tail: 0,
    })
  }

  fn write(&mut self, data: &[u8]) {
    for &byte in data {
      self.mmap[self.head] = byte;
      self.head = (self.head + 1) % self.capacity;
      if self.head == self.tail {
        self.tail = (self.tail + 1) % self.capacity;
      }
    }
  }

  // Aditional methods for reading, seeking, etc., can be added here
}

fn main() -> io::Result<()> {
  let mut ring_buffer = RingBuffer::new("ring_buffer.dat", 1024)?;
  ring_buffer.write(b"Hello, Ring Buffer!");
  Ok(())
}
```
