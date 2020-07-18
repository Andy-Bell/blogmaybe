+++
title = "Jit-in-Rust pt 1"
description = "Very basic start and many errors"
date = 2020-07-17
+++

So hopefully this will end up being a series of blog posts, but no promises (looks at past failure to actually post blogs). This is aiming on documenting my progress in working through [James Coglan's](https://twitter.com/mountain_ghosts) brilliant book [Building Git](https://shop.jcoglan.com/building-git/)

<!-- more -->

### Current progress - makes a blob object, but the hash is wrong

So I took it on myself to not do the easy route with this (follow along in ruby), mainly because ruby is the language I use in my day job (and I like to have mental separation between work and hobby) but also because I wanted to use this as an opportunity to use more rust.

This has caused a bunch of issues / tearing hair out/ fighting ferris to make the borrow checker like my code ([Artists impression](https://pbs.twimg.com/media/DZrHncTVwAAQLJr.jpg:large)), it has also caused me to rethink some assumptions I had made around rust and this project.

Initially my goal was to use no libraries outside rust std. That swiftly changed when I realised there was no easy way to do several key parts: Sha-1 encoding of the data, and Zlib compression.

It has also improved my understanding of how enums and trait objects work. My initial assumption of enums: Creates a type, which can then contain other types, which I can then reference:

```rust
enum MyEnum {
    File: DirEntry,
    Folder: Vec<MyEnum>
}

fn list_files(data: MyEnum) -> () {
    ...
    match abstract_file {
        File => {
            handle_file(abstract_file);
        },
        Folder => {
            list_files(abstract_file);
        }
    }
    ...
}
```

This is close, but not quite right. The actual way it seems to work is that it allows you to create new types which can then be unwrapped to use the internal object:

```rust
enum Files {
    File(DirEntry),
    Folder(Vec<Files>),
}

fn list_files(base_folder: Vec<Files>) -> () {
    ...
    match abstract_file {
        Files::File(file) => {
            handle_file(file);
        },
        Files::Folder(folder) => {
            list_files(folder);
        }
}
```

My misunderstanding with trait objects was due to how Self and trait objects interact. I assumed that if we have a setup like so:

```rust
trait GitObject {
    fn update_oid(&self, oid: String) -> Self
}

impl GitObject for Blob {
    fn update_oid(&self, oid: String) -> Self {
        Blob { /* Fields */ }
    }
}

fn make_some_changes(object: Box<dyn GitObject>) -> () {
    ...
    object.update_oid(oid);
    ...
}
```

That this would be fine, but the compiler started screaming at me when I tried to call update_oid on an object I knew was a Blob, but to the Type checker it only knew was a `dyn GitObject`, as the size of GitObject cannot be known, so I have a current hack for this:

```rust
trait GitObject {
    fn type_name(&self) -> String;
    fn update_oid(&self, oid: String) -> Self;
}
impl Blob {
    pub fn save(object: Box<dyn GitObject>, oid: String) -> Blob {
        Blob { /* Fields */ }
    }
}
impl GitObject for Blob {
    fn type_name(&self) -> String {
        "blob".to_string()
    }
}

fn make_some_changes(object: Box<dyn GitObject>) -> () {
    ...
    if object.type_name() == "blob".to_string() {
        let blob = Blob::save(object);
        save_blob(blob).expect("Error saving blob");
    }
    ...
}
```

Which allows this to work as expected, but is not as clean as I want, as I need to create the object once, then recreate it to update this value, I am sure there is a nicer way to do this (and traits may be incorrect here), but for now it just works(tm).

### Current Issue: Object hashes dont match
So I have this built, and thought I would test it, by creating the Jit .git directory (by init then commit), followed by then listing the objects in the .git/objects directory, wiping it then building an actual git init/commit to regenerate the directory. Assuming everything is working as expected, the folders should almost match (the c git directory would have extra files due to it actually building the tree and commit objects, not just the file blobs). However my current one doesnt work, suggesting that probably the data being passed in through the content function is not building correctly:
```rust
impl Blob {
    pub fn save(object: Box<dyn GitObject>, oid: String) -> Blob {
        let bytes = object.as_ascii();
        let mut out = Vec::new();
        let content = format!("{:?} {:?}\0", object.type_name(), bytes.len());
        out.extend(content.as_bytes());
        out.extend(bytes);
        let hash = Sha1::digest(&*out);
        ...
    }
}
```

So my task tomorrow is to compare how this is building it differently to in the book's ruby version, the easiest way to do so is to build the ruby version to this point, and add logging to both at the generation of the content, and do a side by side comparison of the output to compare them.