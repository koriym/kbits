# I made my own git

**Original:** https://tonystr.net/blog/git_immitation  
**Author:** TonyStr  
**Date:** January 27, 2026

---

Version control used to be a black box for me; I had no idea how files were stored, how diffs were generated or how commits were structured. Since I love reinventing the wheel, why not take a stab at git?

## Hashing

Everything in git is based around hashes, specifically SHA-1 hashes. When you commit a file, git hashes the file and stores it in `.git/objects/`. Then, to be able to find that file again, git makes a "tree" object (a list of files and subdirectories and their corresponding hashes), hashes it, and stores that in `.git/objects/` as well. Then it makes a commit object which contains the tree hash, the previous commit's hash, the author, committer and commit message. This object is also hashed and stored in `.git/objects/`.

My first task was to decide on a hashing algoritm. Git uses SHA-1, which is an old and cryptographically broken algorithm. This doesn't actually matter to me though, since I'll only be using hashes to identify files by their content; not to protect any secrets. But compatibility is not a goal, so why not use the "new" (since 2001) standard, SHA-256 instead.

These objects are also compressed to save space, so writing to and reading from `.git/objects/` will always involve running a compression algoritm. Git uses zlib to compress objects, but looking at competitors, zstd seemed more promising:

> source: https://facebook.github.io/zstd/

Since I don't care about git compatibility, I went for zstd. As for the name, I decided on "tvc", short for "Tony's Version Control" - a simple name which inspires trust 🙂. I would use this name for things like `.tvc` (the equivalent of `.git`) and `.tvcignore` (equivalent of `.gitignore`).

## Implementation

With these prerequisites decided upon, the project seemed quite straight-forward to do. I would just need to do the following steps:

- Read args from stdin
- Read an ignore-file
- Implement ls - print non-ignored files in working directory
- Hash files
- Compress files
- Decompress files
- Generate tree object
- Generate commit objects
- Generate a HEAD file
- Checkout commits

Though I suck at it, my go-to language for side-projects is always Rust. The ls command was quite simple, I just recursively read the current directory, skip any file or directory that matches a path in the ignore file (`.tvcignore`), and run a callback for each file.

```rust
match subcommand.as_str() {
	"ls" => {
		let cb = |path: &Path| {
			let hash = sha256::try_digest(&path)
				.unwrap_or("<invalid hash>".to_string());
			println!("{}\t{}", path.display(), &hash);
		};

		read_dir_recursive(Path::new("./"), &ignore_rules, &cb).unwrap();
	}
}
```

In the callback `cb`, I hash the file contents with `sha256::try_digest(&path)` (I have no idea why hashing is sometimes called digesting, but they seem to be synonymous), and print the path and hash to stdout.

Compression and decompression was also trivial with the zstd library.

```rust
fn decompress_object(object: &str) -> std::io::Result<String> {
    let path = PathBuf::from(format!("./.tvc/objects/{}", object));
    let object = File::open(path)?;

    let mut buf: String = String::new();
    let mut decoder = zstd::Decoder::new(object)?;
    decoder.read_to_string(&mut buf)?;
    decoder.finish();

    Ok(buf)
}

fn compress_file(source: &Path, dest: &Path) -> std::io::Result<()> {
    let input  = File::open(source)?;
    let output = File::create(dest)?;

    let mut encoder = zstd::Encoder::new(output, 3)?;
    std::io::copy(&mut &input, &mut encoder)?;
    encoder.finish()?;

    Ok(())
}
```

## Commits

To generate a commit, you need to store the following properties:

- what type of object is this? (commit)
- what did the file system look like at the time of this commit? (tree)
- which commit came prior to this? (HEAD, if it is set)
- who wrote this commit? (author)
- what message does the commit have?

Git also stores the size of the object in the header, and differentiates between commit author and committer. Since most commits have the same author and committer, I decided to not implement this distinction. This is useful for merges and rebase, but I won't be implementing those features anyway.

```rust
"commit" => {
	let message = args.iter().skip(1).map(|s| s.as_str()).collect::<Vec<_>>().join(" ");
	let author = "god"; // TODO: track commit author
	let parent_hash = head;

	let tree = generate_tree(Path::new("./"), &ignore_rules).expect("tree error");
	let tree_hash = digest(tree);

	/* ===== commit format =====
	<type> \0
	tree\t<tree>\n
	parent\t<parent>\n
	author\t<author>\n
	message\t<message>\n
	*/
	let commit_object = format!("commit \0tree\t{}\nparent\t{}\nauthor\t{}\nmessage\t{}", tree_hash, parent_hash, author, message);
	let commit_hash = digest(&commit_object);
	let dest = PathBuf::from(format!("./.tvc/objects/{}", commit_hash));
	let dest_file = File::create(dest)?;
	let mut encoder = zstd::Encoder::new(dest_file, 3)?;
	
	encoder.write_all(commit_object.as_bytes())?;
	encoder.finish()?;

	// Update HEAD
	fs::write("./.tvc/HEAD", &commit_hash)?;
	println!("HEAD is now at: {}", commit_hash);
}
```

Notice that all the heavy lifting here is done in `generate_tree()`. I decided to not include it's code here, but the implementation is very simple: loop through the root directory, then hash, compress and store each file in `.tvc/objects/`, and add the filename and hash to a string. Subdirectories are simply handled recursively with the same function, and the returned string (a tree object) is hashed, stored, and included in this tree's string.

Any file that is unchanged since last commit, will yield the same hash, and thereby no new file will be written to `.tvc/objects/`. If each commit only touches one of two files, but the repo contains hundreds, then all of those commits will point to the same files for the most part.

Finally, to check out commits, I would have to parse the messy object formats I'd created. My code is quite verbose and messy, but still quite straight-forward. I'm essentially just splitting strings to get the values for commit message, filename & hash, previous commit (called parent in code), etc. Then I store the data in corresponding structs, making it easy to work with the data after parsing.

```rust
#[derive(Debug, Clone)]
struct Commit {
    tree: String,
    parent: String,
    author: String,
    message: String,
}

// Generate the struct from a commit object string
impl From<String> for Commit {
	fn from(string: String) -> Self {
		...
	}
}

#[derive(Debug, Clone)]
struct Tree {
    trees: Vec<(String, Tree)>,		// (path, Tree)
    blobs: Vec<(String, String)>,	// (path, hash)
}

impl Tree {
	fn from_hash(string: &str) -> Self {
		let object = decompress_object(string).unwrap();
		Tree::from_string(object.as_str())
	}
	
	fn from_string(string: &str) -> Self {
		...
	}
}
```

After generating a representation of the objects in memory, I could simply implement another member function for Tree that generates the file structure, and extracts the relevant objects. I was afraid to overwrite my code while testing (as I was testing this on the tvc code), so I made the function take a mandatory path argument to checkout to.

```rust
impl Tree {
	...
	fn generate_fs(&self, path: &Path) -> std::io::Result<()> {
		fs::create_dir(&path).unwrap_or(());

		for (dirname, blob) in &self.blobs {
			let file = decompress_object(blob)?;
			fs::write(path.join(dirname), file.as_bytes())?;
		}

		for (dirname, tree) in &self.trees {
			tree.generate_fs(&path.join(dirname))?;
		}

		Ok(())
	}
}
```

This was a fun exercise. It really drove home the idea that git is just a content-addressable file store (a key-value data store). The hardest part about this project was actually just parsing. If I were to do this again, I would probably use a well-defined language like yaml or json to store object information.

If you want to look at the code, it's available on github.

---

**Tags:** #git #version-control #rust #learning-by-doing

**Curation Note:** Educational deep-dive into Git internals through reimplementation. Excellent for developers who want to understand how version control actually works under the hood.
