# Steganography-Hiding-Information-in-image

from PIL import Image
import numpy as np

def text_to_binary(text):
    return ''.join(format(ord(char), '08b') for char in text)

def binary_to_text(binary):
    chars = [binary[i:i+8] for i in range(0, len(binary), 8)]
    return ''.join(chr(int(char, 2)) for char in chars)

def encode_image(image_path, message, output_path):
    img = Image.open(image_path)
    img = img.convert('RGB')
    data = np.array(img)

    binary_message = text_to_binary(message) + '1111111111111110'  # End delimiter
    msg_index = 0
    rows, cols, _ = data.shape

    for row in range(rows):
        for col in range(cols):
            for channel in range(3):  # R, G, B
                if msg_index < len(binary_message):
                    original_pixel = data[row, col, channel]
                    new_pixel = (original_pixel & ~1) | int(binary_message[msg_index])
                    data[row, col, channel] = new_pixel
                    msg_index += 1

    encoded_img = Image.fromarray(data.astype('uint8'))
    encoded_img.save(output_path)
    print("✅ Message successfully encoded into image.")

def decode_image(image_path):
    img = Image.open(image_path)
    img = img.convert('RGB')
    data = np.array(img)

    binary_message = ''
    for row in data:
        for pixel in row:
            for value in pixel:
                binary_message += str(value & 1)

    end_index = binary_message.find('1111111111111110')
    if end_index != -1:
        binary_message = binary_message[:end_index]
        return binary_to_text(binary_message)
    else:
        return "⚠️ No hidden message found."

# Example usage:
# encode_image('input.png', 'Hello, this is secret!', 'output.png')
# print(decode_image('output.png'))
