```pip install pyinstaller
```
```pyinstaller --onefile --windowed watermark.py
```
```import os
import webbrowser
from tkinter import Tk, filedialog, Label, Button, Entry, StringVar, OptionMenu, colorchooser
from PIL import Image, ImageDraw, ImageFont

# Hàm chọn thư mục đầu vào
def select_folder(var):
    folder = filedialog.askdirectory()
    if folder:
        var.set(folder)

# Hàm chọn file ảnh đóng dấu
def select_image():
    file_path = filedialog.askopenfilename(filetypes=[("Image Files", "*.png;*.jpg;*.jpeg")])
    if file_path:
        watermark_image_var.set(file_path)

# Hàm chọn màu chữ
def choose_color():
    color_code = colorchooser.askcolor(title="Chọn màu chữ")
    if color_code:
        color_var.set(color_code[1])

# Hàm áp dụng watermark
def apply_watermark():
    input_folder = input_folder_var.get()
    output_folder = output_folder_var.get()
    text = text_var.get()
    font_size = int(font_size_var.get())
    position = position_var.get()
    color = color_var.get()
    opacity = int(opacity_var.get())
    watermark_image = watermark_image_var.get()

    if not input_folder or not output_folder or (not text and not watermark_image):
        status_label.config(text="Vui lòng điền đầy đủ thông tin!", fg="red")
        return

    try:
        os.makedirs(output_folder, exist_ok=True)
        font = ImageFont.truetype("arial.ttf", font_size) if text else None

        for filename in os.listdir(input_folder):
            if filename.lower().endswith(("png", "jpg", "jpeg")):
                img_path = os.path.join(input_folder, filename)
                img = Image.open(img_path).convert("RGBA")

                # Tạo layer đóng dấu
                txt_layer = Image.new("RGBA", img.size, (255, 255, 255, 0))
                draw = ImageDraw.Draw(txt_layer)

                if text:
                    # Tính toán kích thước văn bản
                    bbox = draw.textbbox((0, 0), text, font=font)
                    text_width = bbox[2] - bbox[0]
                    text_height = bbox[3] - bbox[1]

                    # Lấy vị trí đóng dấu
                    x, y = get_position(img.size, text_width, text_height, position)

                    # Xử lý màu RGBA
                    rgba_color = tuple(int(color[i:i+2], 16) for i in (1, 3, 5)) + (opacity,)
                    draw.text((x, y), text, font=font, fill=rgba_color)

                if watermark_image:
                    # Thêm watermark hình ảnh
                    watermark = Image.open(watermark_image).convert("RGBA")
                    watermark_width, watermark_height = watermark.size

                    # Lấy vị trí đóng dấu
                    x, y = get_position(img.size, watermark_width, watermark_height, position)

                    # Tạo lớp mờ
                    watermark = watermark.resize((watermark_width, watermark_height), Image.Resampling.LANCZOS)
                    watermark_layer = Image.new("RGBA", watermark.size, (255, 255, 255, opacity))
                    watermark = Image.alpha_composite(watermark, watermark_layer)

                    # Dán watermark lên ảnh
                    txt_layer.paste(watermark, (x, y), mask=watermark)

                # Kết hợp lớp văn bản và ảnh gốc
                combined = Image.alpha_composite(img, txt_layer).convert("RGB")

                # Lưu ảnh đã đóng dấu
                output_path = os.path.join(output_folder, filename)
                combined.save(output_path, "JPEG")
        
        status_label.config(text="Đóng dấu hoàn tất!", fg="green")
    except Exception as e:
        status_label.config(text=f"Lỗi: {e}", fg="red")

# Hàm tính toán vị trí của watermark
def get_position(image_size, element_width, element_height, position):
    img_width, img_height = image_size
    if position == "Trung tâm":
        return (img_width - element_width) // 2, (img_height - element_height) // 2
    elif position == "Góc trên bên trái":
        return 10, 10
    elif position == "Góc trên bên phải":
        return img_width - element_width - 10, 10
    elif position == "Góc dưới bên trái":
        return 10, img_height - element_height - 10
    elif position == "Góc dưới bên phải":
        return img_width - element_width - 10, img_height - element_height - 10
    return 0, 0

# Hàm mở liên kết Donate
def open_donate():
    webbrowser.open("https://www.vietqr.io")  # Thay bằng link donate của bạn

# Giao diện người dùng
root = Tk()
root.title("Phần mềm đóng dấu ảnh hàng loạt")

# Biến lưu trữ
input_folder_var = StringVar()
output_folder_var = StringVar()
text_var = StringVar()
font_size_var = StringVar(value="30")
color_var = StringVar(value="#000000")
opacity_var = StringVar(value="128")
position_var = StringVar(value="Trung tâm")
watermark_image_var = StringVar()

# Giao diện chọn thư mục và cài đặt
Label(root, text="Thư mục chứa ảnh:").grid(row=0, column=0, sticky="w")
Entry(root, textvariable=input_folder_var, width=50).grid(row=0, column=1)
Button(root, text="Chọn", command=lambda: select_folder(input_folder_var)).grid(row=0, column=2)

Label(root, text="Thư mục xuất ảnh:").grid(row=1, column=0, sticky="w")
Entry(root, textvariable=output_folder_var, width=50).grid(row=1, column=1)
Button(root, text="Chọn", command=lambda: select_folder(output_folder_var)).grid(row=1, column=2)

Label(root, text="Nội dung đóng dấu:").grid(row=2, column=0, sticky="w")
Entry(root, textvariable=text_var, width=50).grid(row=2, column=1, columnspan=2)

Label(root, text="Kích thước chữ:").grid(row=3, column=0, sticky="w")
Entry(root, textvariable=font_size_var, width=10).grid(row=3, column=1, sticky="w")

Label(root, text="Màu chữ:").grid(row=4, column=0, sticky="w")
Button(root, text="Chọn màu", command=choose_color).grid(row=4, column=2)
Entry(root, textvariable=color_var, width=10).grid(row=4, column=1, sticky="w")

Label(root, text="Độ mờ (0-255):").grid(row=5, column=0, sticky="w")
Entry(root, textvariable=opacity_var, width=10).grid(row=5, column=1, sticky="w")

Label(root, text="Vị trí đóng dấu:").grid(row=6, column=0, sticky="w")
OptionMenu(root, position_var, "Trung tâm", "Góc trên bên trái", "Góc trên bên phải", 
           "Góc dưới bên trái", "Góc dưới bên phải").grid(row=6, column=1, sticky="w")

Label(root, text="Hình ảnh đóng dấu:").grid(row=7, column=0, sticky="w")
Entry(root, textvariable=watermark_image_var, width=50).grid(row=7, column=1)
Button(root, text="Chọn", command=select_image).grid(row=7, column=2)

Button(root, text="Bắt đầu đóng dấu", command=apply_watermark).grid(row=8, column=1, pady=10)

# Nút Donate
Button(root, text="Donate", command=open_donate, bg="orange", fg="white").grid(row=9, column=1, pady=10)

# Tên tác giả
Label(root, text="Tác giả: Nguyễn Văn A", fg="blue").grid(row=10, column=1, pady=10)

status_label = Label(root, text="")
status_label.grid(row=11, column=0, columnspan=3)

root.mainloop()
```
