import spotipy
from spotipy.oauth2 import SpotifyOAuth
import re
import tkinter as tk
from tkinter import ttk, messagebox

# Spotify API credentials 
SPOTIFY_CLIENT_ID = 'yourclientid'
SPOTIFY_CLIENT_SECRET = 'yourclientsecret'
SPOTIFY_REDIRECT_URI = 'http://localhost:8000/callback/'

# Authenticate with Spotify
sp = spotipy.Spotify(auth_manager=SpotifyOAuth(
    client_id=SPOTIFY_CLIENT_ID,
    client_secret=SPOTIFY_CLIENT_SECRET,
    redirect_uri=SPOTIFY_REDIRECT_URI,
    scope="playlist-modify-public playlist-read-private"
))

def get_playlist_id(playlist_url):
    """Extracts the playlist ID from the Spotify playlist URL."""
    match = re.match(r"https://open\.spotify\.com/playlist/(\w+)", playlist_url)
    if match:
        return match.group(1)
    else:
        raise ValueError("Invalid Spotify playlist URL")

def get_all_album_tracks(album_id):
    """Fetches all tracks from a given album."""
    results = sp.album_tracks(album_id)
    tracks = results['items']
    while results['next']:
        results = sp.next(results)
        tracks.extend(results['items'])
    return [(track['id'], track['name'], track['artists'][0]['name']) for track in tracks]

def get_tracks_from_playlist(playlist_url):
    """Fetches all tracks from albums in a given playlist, preserving album order based on when tracks were added."""
    playlist_id = get_playlist_id(playlist_url)
    playlist_details = sp.playlist(playlist_id)
    playlist_name = playlist_details['name']

    # Fetch all items from the playlist
    playlist_items = []
    results = sp.playlist_items(playlist_id)
    playlist_items.extend(results['items'])
    while results['next']:
        results = sp.next(results)
        playlist_items.extend(results['items'])

    # Map albums to the earliest added_at timestamp of their tracks
    album_order = {}
    for item in playlist_items:
        track = item['track']
        album_id = track['album']['id']
        added_at = item['added_at']
        if album_id not in album_order or added_at < album_order[album_id]:
            album_order[album_id] = added_at

    # Sort albums by their earliest added_at timestamp
    sorted_album_ids = sorted(album_order, key=album_order.get)

    # Retrieve tracks from each album in sorted order
    all_tracks = []
    for album_id in sorted_album_ids:
        album_tracks = get_all_album_tracks(album_id)
        all_tracks.extend(album_tracks)

    return all_tracks, playlist_name

def create_playlist_with_tracks(tracks, original_playlist_name):
    """Creates a new playlist with tracks and the name '[Original playlist name] (Albums)'."""
    new_playlist_name = f"{original_playlist_name} (Albums)"
    user_id = sp.current_user()['id']
    new_playlist = sp.user_playlist_create(user=user_id, name=new_playlist_name, public=True)
    new_playlist_id = new_playlist['id']

    track_ids = [track[0] for track in tracks]
    for i in range(0, len(track_ids), 100):
        sp.playlist_add_items(new_playlist_id, track_ids[i:i+100])

    messagebox.showinfo("Success", f"New playlist '{new_playlist_name}' created successfully!")

def display_tracks(tracks, playlist_name):
    """Displays the tracks in a GUI."""
    def copy_to_clipboard():
        clipboard_text = "\n".join([f"https://open.spotify.com/track/{track[0]}" for track in tracks])
        root.clipboard_clear()
        root.clipboard_append(clipboard_text)
        root.update()
        messagebox.showinfo("Success", "Track links copied to clipboard!")

    def create_playlist():
        create_playlist_with_tracks(tracks, playlist_name)

    root = tk.Tk()
    root.title("Tracks from Playlist")

    frame = ttk.Frame(root, padding="10")
    frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

    scrollbar = ttk.Scrollbar(frame, orient=tk.VERTICAL)
    scrollbar.grid(row=0, column=1, sticky=(tk.N, tk.S))

    track_list = tk.Listbox(frame, yscrollcommand=scrollbar.set, width=80, height=20)
    for track_id, track_name, artist_name in tracks:
        track_list.insert(tk.END, f"{track_name} by {artist_name}")
    track_list.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

    scrollbar.config(command=track_list.yview)

    copy_button = ttk.Button(root, text="Copy to Clipboard", command=copy_to_clipboard)
    copy_button.grid(row=1, column=0, pady=5)

    create_button = ttk.Button(root, text="Create Playlist", command=create_playlist)
    create_button.grid(row=2, column=0, pady=5)

    root.mainloop()

def main():
    def fetch_tracks():
        playlist_url = url_entry.get()
        try:
            tracks, playlist_name = get_tracks_from_playlist(playlist_url)
            display_tracks(tracks, playlist_name)
        except Exception as e:
            messagebox.showerror("Error", f"{e}")

    root = tk.Tk()
    root.title("Spotify Playlist Tracks")

    frame = ttk.Frame(root, padding="10")
    frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

    label = ttk.Label(frame, text="Enter Spotify Playlist URL:")
    label.grid(row=0, column=0, sticky=tk.W, pady=5)

    url_entry = ttk.Entry(frame, width=50)
    url_entry.grid(row=1, column=0, pady=5, sticky=tk.W)

    fetch_button = ttk.Button(frame, text="Fetch Tracks", command=fetch_tracks)
    fetch_button.grid(row=2, column=0, pady=10, sticky=tk.W)

    root.mainloop()

if __name__ == "__main__":
    main()
