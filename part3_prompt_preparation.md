# Part 3: Prompt Preparation

## Task 3.1: Comprehensive Prompt Documentation

**Selected PR:** **PR #3145 (Fix `import --flat` with `--move`)**
**Repository:** `beetbox/beets`

### 3.1.1 Repository Context
`beets` is a sophisticated command-line media library management system written in Python. It is designed for audiophiles who need to organize large collections of music files with precision.

* **Core Function:** It acts as an "intelligent gatekeeper." Users point it at a directory of messy audio files, and Beets identifies the music using the MusicBrainz database, fetches correct metadata (tags), and then renames and moves the files into a structured library folder.
* **Target Users:** Power users, archivists, and DJs who prefer command-line tools over graphical interfaces like iTunes.
* **Architecture:** The system uses a plugin-based architecture around a core library database (SQLite). The `importer` module is the heart of the system, handling the complex logic of file manipulation, duplicate detection, and path generation.

### 3.1.2 Pull Request Description
This Pull Request addresses a logic error in the file organization module where two user configuration options were conflicting.

* **The Issue:** The system provides two distinct configuration flags: `move` (which relocates files from the source to the library) and `flat` (which disables directory nesting, keeping all files in one single folder). A bug existed where the `move` logic unconditionally applied the default directory template (Artist/Album), effectively ignoring the user's request for a `flat` structure.
* **The Fix:** The PR modifies the destination path calculation logic. It introduces a check to see if the `flat` flag is active. If it is, the system now bypasses the directory generation and constructs a path directly in the library root.
* **Impact:** This ensures that the system honors the user's configuration intent. If a user asks for a flat library, they get a flat library, even when moving files.

### 3.1.3 Acceptance Criteria
1.  ✓ **When** the user runs an import with `import.move=True` AND `import.flat=True`, the imported files **should** be moved directly to the library root directory (e.g., `~/Music/Song.mp3`).
2.  ✓ **The implementation should** ensure that NO subdirectories (like `/Artist/Album/`) are created during this specific operation.
3.  ✓ **When** `import.flat=False` (default), the system **should** continue to create the standard Artist/Album hierarchy.
4.  ✓ **The implementation should** handle the `copy` operation identically to `move`: if `copy=True` and `flat=True`, the file should be copied to the root.
5.  ✓ **When** a file is moved to the root, the filename **should** still be sanitized and formatted according to the user's naming template (e.g., preserving Track Numbers if configured).

### 3.1.4 Edge Cases
1.  **Filename Collisions:** In a flat directory, the risk of two songs having the same name (e.g., `01 - Intro.mp3`) is extremely high. The prompt must ensure that existing conflict resolution logic (appending unique numbers like `.1`) is applied to the root directory.
2.  **Cross-Device Moves:** If the source file is on an external drive and the library is on the internal drive, `os.rename` typically fails. The solution relies on `shutil.move`, which handles this, but the model should be aware of this distinction.
3.  **Permissions:** The model should consider that writing to the root of a library path might require different permissions than creating a new subfolder, although this is typically handled by the OS.

### 3.1.5 Initial Prompt

```python
You are a Senior Python Developer working on the `beets` music library manager.

**Context:**
We have identified a bug in the `importer` module. Users are reporting that the `--flat` configuration (which should prevent directory nesting) is being ignored when the `--move` or `--copy` option is enabled. The system is forcing the creation of Artist/Album directories against the user's wishes.

**Task:**
Modify `beets/importer.py` to fix this logic error. You need to ensure that the destination path calculation respects the `flat` flag.

**Technical Requirements:**
1.  **Locate the Logic:** Find the `ImportTask` class, specifically the method responsible for calculating the destination path (likely `destination()` or `manipulate_files()`).
2.  **Implement the Check:** Add a conditional branch.
    * IF `config['import']['flat']` is True: The destination path should be `os.path.join(lib_dir, filename)`.
    * ELSE: The destination path should use the standard logic (Artist/Album/...).
3.  **Preserve Functionality:** Ensure that file renaming (sanitization) still happens. We only want to skip the *directory creation*, not the filename formatting.

**Testing Requirements:**
We need a regression test to prevent this from breaking again. Please write a test case in `test/test_importer.py` that:
1.  Configures the test environment with `import_move=True` and `import_flat=True`.
2.  Simulates importing a music file.
3.  Asserts that the file exists at the Library Root path.
4.  Asserts that **no** subdirectories (Artist/Album) were created.

**Output:**
Provide the corrected method code for `importer.py` and the complete test case class.
