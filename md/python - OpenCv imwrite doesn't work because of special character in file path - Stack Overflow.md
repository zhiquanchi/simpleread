> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [stackoverflow.com](https://stackoverflow.com/questions/44330084/opencv-imwrite-doesnt-work-because-of-special-character-in-file-path)

> I can't save an image when the file path has special character (like "é" for example). Here is a test from Python 3 shell : >>> cv2.imwrite('gel/test.jpg', frame) True >>> cv2.i...

![](https://lh3.googleusercontent.com/a/ACg8ocIt31FS_9gZ7_NWXrbzYof1w2n7M6_SflmR_tNlBrrpIA=k-s32)TeeVy

I can't save an image when the file path has special character (like"é" for example).

Here is a test from Python 3 shell :

```
>>> cv2.imwrite('gel/test.jpg', frame)
True
>>> cv2.imwrite('gel/ééé/test.jpg', frame)
False
>>> cv2.imwrite('gel/eee/test.jpg', frame)
True
```

Any ideas how to do it?

Thanks!

**EDIT :**

Unfortunately, all suggestions proposed by @PM2Ring and @DamianLattenero don't seem to work :(

So, I use the @cdarke's solution, here's my final code :

```
destination = 'gel/ééé/'
gel = 'test.jpg'
script_path = os.getcwd()
os.chdir(destination)
cv2.imwrite(gel, frame)
os.chdir(script_path)
```

![](https://lh6.googleusercontent.com/-Y01fMf7ERU4/AAAAAAAAAAI/AAAAAAAABV4/1nXyivK6ElI/photo.jpg?sz=64)jdhao

You can first encode the image with OpenCV and then save it with numpy `tofile()` method since the encoded image is a one-dim numpy ndarray：

```
is_success, im_buf_arr = cv2.imencode(".jpg", frame)

im_buf_arr.tofile('gel/ééé/test.jpg')
```

![](https://i.stack.imgur.com/zto2h.jpg?s=64&g=1)Parham

These are my functions for both writing and reading:

```
def read_image(path):
    return cv2.imdecode(np.fromfile(path,dtype=np.uint8), -1)

def save_image(img, path):
    cv2.imencode(".png", img)[1].tofile(path+'/your_file_name.png')
```

![](https://i.stack.imgur.com/S9pH7.jpg?s=64&g=1)developer_hatch

Try encoding with:

```
cv2.imwrite('gel/ééé/test.jpg'.encode('utf-8'), frame) # or just .encode(), 'utf-8' is the default
```

If you are using windows, maybe with:

```
cv2.imwrite("gel/ééé/test.jpg".encode("windows-1252"), frame)
```

Or now reading the @PM user answer, according your utf-16 windows:

```
cv2.imwrite("gel/ééé/test.jpg".encode('UTF-16LE'), frame)
```

if non of that works to you, try this:

```
ascii_printable = set(chr(i) for i in range(0x20, 0x7f))

def convert(ch):
    if ch in ascii_printable:
        return ch
    ix = ord(ch)
    if ix < 0x100:
        return '\\x%02x' % ix
    elif ix < 0x10000:
        return '\\u%04x' % ix
    return '\\U%08x' % ix

path = 'gel/ééé/test.jpg'

converted_path = ''.join(convert(ch) for ch in 'gel/ééé/test.jpg')

cv2.imwrite(converted_path, frame)
```

![](https://i.stack.imgur.com/d7vxz.png?s=64&g=1)Ramius

In one line.

`cv2.imencode(".jpg",frame)[1].tofile("gel/ééé/test.jpg")`

![](https://i.stack.imgur.com/P1KCj.jpg?s=64&g=1)hct

Try this

```
img_encoded = cv2.imencode(".jpg",frame)[1]
with open("ééé/test.jpg", "wb") as f:
    f.write(img_encoded)
```

![](https://www.gravatar.com/avatar/8d9e9d363c161a22939778334a2b6e12?s=64&d=identicon&r=PG&f=y&so-version=2)Víctor Quilón

Encoding issues.... difficult but no impossible

if you have this string = `'テスト/abc.jpg'`

**You can encode as Windows encoding the characters like this**->

`print('テスト/abc.jpg'.encode('utf-8').decode('unicode-escape'))`

And you get something like this = `'ãã¹ã/abc.jpg'`

**Then if you want to read the file and get the filenames readable and usable, you can use some library to read the filenames of your path and then change the encoding**->

`#fname is like 'ãã¹ã/abc.jpg'`

`fname.encode('iso-8859-1').decode('utf-8'))` # This result of your initial string =`'テスト/abc.jpg'`