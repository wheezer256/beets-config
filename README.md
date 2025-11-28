# Personal Beets Configuration

My configuration for [beets](https://beets.io/) (v2.0+), designed to organize a large music library while **strictly preserving original file modification times** (mtime).

## 📦 Features

* **Modular Config:** Split into `config.yaml`, `plugins.yaml`, `local.yaml`, and `secrets.yaml`.
* **Modern Plugins:** Uses `filetote` (replaces `copyartifacts`) and `beetcamp` (Bandcamp support).
* **Metadata Sources:** MusicBrainz (Primary), Discogs, and Bandcamp.
* **Folder Structure:** Groups artists by first letter (e.g., `M/Metallica/Master of Puppets`), handling "The" and special characters automatically.
* **Preservation Workflow:** A custom bash strategy to update tags without resetting file modification timestamps.

## 🛠 Installation

This setup uses `pipx` to run Beets in an isolated environment.

1.  **Install Beets**
    ```bash
    pipx install beets
    ```

2.  **Inject Dependencies & Plugins**
    We need to inject the external plugins into the Beets virtual environment.
    ```bash
    # Install Filetote (moves non-music files) and Beetcamp (Bandcamp support)
    pipx inject beets beets-filetote beetcamp

    # Ensure core dependencies are up to date
    pipx runpip beets install --upgrade discogs-client requests
    ```

## 📂 Configuration Structure

* `config.yaml`: The main hub. Imports the other files and sets matching rules.
* `plugins.yaml`: List of enabled plugins.
* `secrets.yaml`: **(Ignored)** Contains API keys for Discogs/MusicBrainz.
* `local.yaml`: **(Ignored)** Contains local file paths and logging settings.

### Setup
1.  Clone this repo to `~/.config/beets/`.
2.  Set up your secrets:
    ```bash
    cp secrets.example.yaml secrets.yaml
    # Edit secrets.yaml with your actual API tokens
    ```
3.  Create a `local.yaml` with your machine-specific paths:
    ```yaml
    directory: ~/Music
    library: ~/.config/beets/library.db
    import:
        log: ~/beets_import.log # Logs to parent directory
    ui:
        color: yes
    ```

## ⏳ The "Preserve Dates" Workflow

By default, Beets updates file modification times whenever it writes tags. To keep the original "Date Modified" (e.g., 1999) while still organizing the files and updating the internal metadata, use this two-step process.

### Step 1: Import Without Writing
Run the import with the `-W` (no write) flag. This identifies the music, moves it to the correct `A/Artist/Album` folder, and updates the database, but **does not touch the file content**.

```bash
beet import -W /path/to/incoming/music
```

### Step 2: "Time-Travel" Tag Writing

To update the internal tags (ID3/FLAC) while restoring the old timestamp, run this script on your library folder:

# Navigate to your library
```bash
cd ~/Music
```

# Run the loop
```find . -type f \( -name "*.mp3" -o -name "*.flac" -o -name "*.m4a" \) | while read filename; do
    # 1. Save timestamp
    touch -r "$filename" "$filename.bak_time"

    # 2. Force Beets to write tags
    beet write -f "$filename"

    # 3. Restore timestamp
    touch -r "$filename.bak_time" "$filename"

    # 4. Cleanup
    rm "$filename.bak_time"
    echo "Updated tags & restored time: $filename"
done
```
