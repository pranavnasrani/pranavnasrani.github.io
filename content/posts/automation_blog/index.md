---
title: Setting up the Blog
date: 2024-12-07
draft: false
description: Trying to automate the task of putting blogs into hugo site template
tags:
  - hugo
  - developers
  - obsidian
  - python
---
I don't know why i'm making this blog, except for the sole reason of it also acting as my portfolio. However after seeing NetworkChuck's video on his blog pipeline [see here](https://www.youtube.com/watch?v=dnE7c0ELEH8), I wanted to do something similar. However I wanted to make it even more seamless than typing the blogs out in Obsidian then using some script to publish it. I wanted to make a GUI python script (since python is so easy to work with). Read on to see how I did that

This blog will be a record of all my accomplishments and any other things I find worthy of sharing with the world.

I have also integrated comments and likes and views into this blog system.


**Trying to automate the task of putting blogs into hugo site template**
Hey so i'm developing a program that will automate the task of taking my notes from obsidian and putting them on the Hugo/Blowfish website. Till now I had to do it manually using terminal but i've been working on a tkinter python program (cause python is so easy to work with)
**Trying to automate the task of putting blogs into hugo site template**


This was my first attempt at it its quite simple and just correctly formats obsidian's markdown files and places them in the site folder without doing any git/github sync and pushes.
!!![Image Description](Pasted%20image%2020241207185943.png)


**The final product**
After a few hours I finally included all the desired functionality and it looks something like this:
- github pull/push
- featured image
- listing all posts

!!![Image Description](Pasted%20image%2020241208130848.png)
Its neat and convenient and the code is provided below:

```python
import os

import re

import shutil

from tkinter import Tk, Label, Entry, Button, filedialog, messagebox, Listbox, Scrollbar, Frame

from tkinterdnd2 import TkinterDnD, DND_FILES

import subprocess

from PIL import Image

import json

  

# Global variables

featured_image_path = None

dropped_files = []

config_file = "config.json"

paths = {}

  

def load_or_set_paths():

    global paths

    if os.path.exists(config_file):

        with open(config_file, "r") as file:

            paths = json.load(file)

    else:

        user_documents = filedialog.askdirectory(title="Select Your Documents Directory")

        obsidian_vault = filedialog.askdirectory(title="Select Your Obsidian Vault Directory")

        site_repo = filedialog.askdirectory(title="Select Your Hugo Site Repository Directory")

  

        paths = {

            "documents": user_documents,

            "obsidian_vault": obsidian_vault,

            "site_repo": site_repo

        }

  

        with open(config_file, "w") as file:

            json.dump(paths, file)

  

def process_files():

    global featured_image_path, dropped_files

    target_folder_name = target_folder_entry.get()

    if not target_folder_name.strip():

        messagebox.showerror("Error", "Please enter a folder name.")

        return

  

    # Define target folder path

    target_folder = os.path.join(paths["documents"], "pranavasranisite", "content", "posts", target_folder_name)

  

    # Check if the target folder exists and replace it

    if os.path.exists(target_folder):

        shutil.rmtree(target_folder)  # Remove the existing folder

  

    # Create a new folder

    os.makedirs(target_folder, exist_ok=True)

  

    first_image_handled = False  # Flag to handle the first image duplication

  

    for filepath in dropped_files:

        filename = os.path.basename(filepath)

  

        if filename.endswith(".md"):

            # Process Markdown files

            with open(filepath, "r", encoding="utf-8") as file:

                content = file.read()

  

            # Find and replace image links

            images = re.findall(r'\[\[([^]]*\.png)\]\]', content)

            for image in images:

                # Preserve the original filename, ensuring spaces are URL encoded as %20

                encoded_image = image.replace(' ', '%20')

                markdown_image = f"!![Image Description]({encoded_image})"

                content = content.replace(f"[[{image}]]", markdown_image)

  

                # Handle the first image duplication

                if not first_image_handled:

                    source_image_path = os.path.join(os.path.dirname(filepath), image)

                    if os.path.exists(source_image_path):

                        target_image_path = os.path.join(target_folder, image)

                        shutil.copy(source_image_path, target_image_path)  # Copy the original image

                        first_image_handled = True

  

            # Write updated content to new Markdown file in target folder

            target_md_path = os.path.join(target_folder, "index.md")

            with open(target_md_path, "w", encoding="utf-8") as file:

                file.write(content)

  

        elif filename.endswith((".png", ".jpg", ".jpeg")):

            # Copy image files to target folder

            shutil.copy(filepath, target_folder)

  

    # Handle featured image

    if featured_image_path and os.path.exists(featured_image_path):

        featured_target_path = os.path.join(target_folder, "featured.png")

        # Convert to PNG if not already

        if not featured_image_path.lower().endswith('.png'):

            img = Image.open(featured_image_path)

            img.save(featured_target_path, 'PNG')

        else:

            shutil.copy(featured_image_path, featured_target_path)

  

    messagebox.showinfo("Success", f"Files processed and saved to:\n{target_folder}")

  

    # Reset featured image and list of dropped files after processing

    dropped_files = []

    featured_image_path = None

    dropped_files_label.config(text="Drop files here")

    featured_image_label.config(text="No image selected")

  

def select_featured_image():

    global featured_image_path

    initial_dir = paths["obsidian_vault"]

    filetypes = [

        ("PNG files", "*.png"),

        ("JPEG files", "*.jpg *.jpeg"),

        ("All image files", "*.png *.jpg *.jpeg")

    ]

    featured_image_path = filedialog.askopenfilename(

        initialdir=initial_dir,

        title="Select Featured Image",

        filetypes=filetypes

    )

    if featured_image_path:

        featured_image_label.config(text=f"Selected: {os.path.basename(featured_image_path)}")

    else:

        featured_image_label.config(text="No image selected")

  

def on_drop(event):

    global dropped_files

    dropped_files = event.data.split(" ")

    dropped_files_label.config(text="\n".join(dropped_files))

  

def browse_files():

    global dropped_files

    initial_dir = paths["obsidian_vault"]

    filetypes = [("All files", "*.*")]

    selected_files = filedialog.askopenfilenames(

        initialdir=initial_dir,

        title="Select Markdown or Image Files",

        filetypes=filetypes

    )

    # Update dropped_files and label

    dropped_files.extend(selected_files)

    dropped_files_label.config(text="\n".join(dropped_files))

  

def push_to_github():

    try:

        # Set the path to the site repository

        site_path = paths["site_repo"]

        # Change to the repository directory

        os.chdir(site_path)

        # Run Git commands

        commands = [

            ["git", "add", "."],

            ["git", "commit", "-m", "Site update"],

            ["git", "pull"],

            ["git", "push", "-u", "origin", "master"]

        ]

        # Execute each command and capture output

        for cmd in commands:

            result = subprocess.run(cmd, capture_output=True, text=True, check=True)

            print(f"Command {' '.join(cmd)} output: {result.stdout}")

        # Show success message

        messagebox.showinfo("GitHub Push", "Successfully pushed to GitHub!")

    except subprocess.CalledProcessError as e:

        # Handle Git command errors

        error_message = f"Git error: {e.stderr}"

        messagebox.showerror("GitHub Push Error", error_message)

    except Exception as e:

        # Handle other potential errors

        messagebox.showerror("Error", str(e))

  

def load_posts():

    posts_folder = os.path.join(paths["documents"], "pranavasranisite", "content", "posts")

    # Print the actual path to debug

    print(f"Looking for posts in: {posts_folder}")

    # Check if the posts folder exists

    if os.path.exists(posts_folder):

        # Try to list directories inside the posts folder

        posts = [f for f in os.listdir(posts_folder) if os.path.isdir(os.path.join(posts_folder, f))]

        # Debugging: Print the list of posts found

        if posts:

            print(f"Found the following posts: {posts}")

        else:

            print("No post folders found.")

        # Clear previous list and load new posts

        post_listbox.delete(0, 'end')  # Clear previous list

        for post in posts:

            post_listbox.insert('end', post)  # Add each post folder to the listbox

  

    else:

        print(f"Post folder does not exist at the path: {posts_folder}")

        messagebox.showerror("Error", "Posts folder not found.")

  

def delete_post():

    selected_post = post_listbox.curselection()

    if not selected_post:

        messagebox.showerror("Error", "Please select a post to delete.")

        return

    post_name = post_listbox.get(selected_post)

    posts_folder = os.path.join(paths["documents"], "pranavasranisite", "content", "posts", post_name)

  

    # Confirm deletion

    if messagebox.askyesno("Confirm Deletion", f"Are you sure you want to delete the post: {post_name}?"):

        shutil.rmtree(posts_folder)  # Delete the post folder

        load_posts()  # Reload the post list

  

# Initialize the GUI

load_or_set_paths()

  

root = TkinterDnD.Tk()

root.title("Markdown and Image Processor with Post Manager")

root.geometry("800x800")

  

# Main layout frame

main_frame = Frame(root)

main_frame.pack(fill="both", expand=True, padx=10, pady=10)

  

# Left column for instructions, file dropping, and target folder

left_frame = Frame(main_frame)

left_frame.grid(row=0, column=0, sticky="n")

  

# Instructions

Label(left_frame, text="Drag and drop Markdown and image files below:").pack(pady=10)

  

# Drop target area

dropped_files_label = Label(left_frame, text="Drop files here", bg="lightgray", relief="sunken", width=50, height=10)

dropped_files_label.pack(pady=10)

dropped_files_label.drop_target_register(DND_FILES)

dropped_files_label.dnd_bind("<<Drop>>", on_drop)

  

# Browse files button

browse_button = Button(left_frame, text="Browse Files", command=browse_files)

browse_button.pack(pady=5)

  

# Target folder name input

Label(left_frame, text="Enter new post folder:").pack(pady=10)

target_folder_entry = Entry(left_frame, width=40)

target_folder_entry.pack(pady=5)

  

# Featured Image section

Label(left_frame, text="Featured Image:").pack(pady=5)

  

# Featured image selection button

featured_image_button = Button(left_frame, text="Select Featured Image", command=select_featured_image)

featured_image_button.pack(pady=5)

  

# Label to show selected featured image

featured_image_label = Label(left_frame, text="No image selected")

featured_image_label.pack(pady=5)

  

# Right column for actions and post management

right_frame = Frame(main_frame)

right_frame.grid(row=0, column=1, sticky="n")

  

# Process button

process_button = Button(right_frame, text="Process Files", command=process_files)

process_button.pack(pady=20)

  

# GitHub Push button

github_push_button = Button(right_frame, text="Push to GitHub", command=push_to_github)

github_push_button.pack(pady=20)

  

# Post management section

Label(right_frame, text="Manage Posts:").pack(pady=10)

  

# Listbox to display posts

post_listbox = Listbox(right_frame, width=50, height=10)

post_listbox.pack(pady=10)

  

# Scrollbar for the listbox

scrollbar = Scrollbar(right_frame, orient="vertical", command=post_listbox.yview)

scrollbar.pack(side="right", fill="y")

post_listbox.config(yscrollcommand=scrollbar.set)

  

# Load posts button

load_button = Button(right_frame, text="Load Posts", command=load_posts)

load_button.pack(pady=5)

  

# Delete post button

delete_button = Button(right_frame, text="Delete Selected Post", command=delete_post)

delete_button.pack(pady=5)

  

# Start the GUI event loop

root.mainloop()
```

![Download exe for Windows for automation script](blog.exe)