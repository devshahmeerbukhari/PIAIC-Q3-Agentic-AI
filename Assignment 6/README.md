# Assignment 06-Exploring LLM Models for Creative Video Generation and Script Analysis
## Muhammad Shahmeer Bukhari

import openai
import moviepy.editor as mpy
import os
from PIL import Image
from io import BytesIO

# Function to analyze a script using an LLM
def analyze_script(script_text):
    print("Analyzing the script...")
    analysis_prompt = (
        "You are a creative assistant. Analyze the following script for key themes, emotions, and "
        "visualization suggestions:\n\n" + script_text + 
        "\n\nPlease provide a detailed analysis with suggested visuals and tone.")

    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are an expert in creative writing and media."},
            {"role": "user", "content": analysis_prompt},
        ]
    )

    return response['choices'][0]['message']['content']

# Function to generate images based on descriptions
def generate_images(descriptions):
    images = []
    for i, description in enumerate(descriptions):
        print(f"Generating image {i + 1} based on: {description}")
        response = openai.Image.create(
            prompt=description,
            n=1,
            size="512x512"
        )
        image_url = response['data'][0]['url']

        # Download and save the image locally
        image_data = BytesIO(requests.get(image_url).content)
        image = Image.open(image_data)
        image.save(f"image_{i + 1}.png")
        images.append(f"image_{i + 1}.png")

    return images

# Function to create a video from generated images and text
def create_video(images, script_text, output_file="output_video.mp4"):
    print("Creating video...")
    clips = []
    
    for img in images:
        # Create a clip for each image
        image_clip = mpy.ImageClip(img, duration=5)  # Each image shown for 5 seconds
        clips.append(image_clip)

    # Concatenate image clips
    video = mpy.concatenate_videoclips(clips, method="compose")

    # Add script text as subtitles
    video = video.set_duration(len(clips) * 5)
    video = video.set_audio(None)  # No audio for now

    # Write video to file
    video.write_videofile(output_file, fps=24)

    print(f"Video saved to {output_file}")
    return output_file

# Main function
def main():
    script_text = """
    Your script goes here. Replace this placeholder with the actual content.
    """

    # Step 1: Analyze the script
    analysis = analyze_script(script_text)
    print("Script Analysis:", analysis)

    # Step 2: Extract visualization suggestions
    visualization_suggestions = [
        "A sunny day in a bustling city street",
        "A serene countryside with a flowing river",
        "A close-up of a thoughtful person writing in a journal",
    ]  # Replace with parsed suggestions from analysis

    # Step 3: Generate images for the video
    images = generate_images(visualization_suggestions)

    # Step 4: Create video from the generated images and script
    create_video(images, script_text)

if __name__ == "__main__":
    main()
    

     