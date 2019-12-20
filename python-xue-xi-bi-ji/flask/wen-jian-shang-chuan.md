# 文件上传

#### 文件上传：

* **enctype：**在HTML中的form表单中form标签默认是\`enctype="application/x-www-form-urlencoded"\`，在文件上传时需要设置为\`enctype="multipart/form-data"\`，不然文件上传不会成功。
* **后台获取上传的文件：**fileobj = request.files.get\('input\_file\_name'\)，需要注意的是，get方法的参数是HTML中文件input标签指定的name属性值，而不是上传的文件名称。
* **文件名处理：**使用fileobj.filename即可获取到文件名，但是不建议直接使用这个文件名，为了安全考虑，建议使用from werkzeug.utils import secure\_filename对文件名进行过滤处理一下。
* **保存文件：**使用返回的文件对象的save方法即可，fileobj.save\(file\_path\)，file\_path是保存文件的绝对路径。
* **后台发送文件到浏览器：**使用from flask import send\_from\_directory，直接返回对应的文件即可，send\_from\_directory需要两个参数，第一个参数是文件所在目录，第二个参数是文件名。

#### 文件验证：

* **Form验证：**是使用from wtforms import Form的子类进行验证。
* **字段类型：**from wtforms import FileField，FileField表示文件类型。
* **验证器：**from flask\_wtf.file import FileRequired, FileAllowed，FileRequired表示文件不能为空，FileAllowed表示文件的后缀名类型。
* **多元素结合：**request中有文件和文本等多种类型的元素时，从request中获取数据的方式也不同时，比如：request.form和request.files，再想要使用Form表单对象进行验证时，就需要使用from werkzeug.datastructures import CombinedMultiDict将多种元素结合起来，再传入表单对象进行验证。
* **数据获取：**经过表单对象验证过后，可以通过“form.\[attr\_name\].data”的方式获取文件和文本等数据，这种方式和通过request获取数据是一样的。

**简单示例：**

**HTML文件upload.html主要代码**

```html
<form action="" method="post" enctype="multipart/form-data">
        <table>
            <tbody>
                <tr>
                    <td>头像：</td>
                    <td><input type="file" name="avatar"></td>
                </tr>
                <tr>
                    <td>描述：</td>
                    <td><input type="text" name="desc"></td>
                </tr>
                <tr>
                    <td></td>
                    <td><input type="submit" value="提交"></td>
                </tr>
            </tbody>
        </table>
    </form>
```

**浏览器效果**

![](/assets/flask-12.png)

**表单对象文件forms.py**

```py
from wtforms import Form, FileField, StringField
from wtforms.validators import InputRequired
from flask_wtf.file import FileRequired, FileAllowed


class UploadFileForm(Form):
    # FileField表示字段为文件类型
    avatar = FileField(validators=[FileRequired(), FileAllowed(['jpg', 'png', 'gif'])])
    # StringField表示字段为字符串类型
    desc = StringField(validators=[InputRequired()])
```

**主py文件**

```py
import os
from werkzeug.utils import secure_filename
from werkzeug.datastructures import CombinedMultiDict
from flask import Flask, request, render_template, send_from_directory
from forms import UploadFileForm

app = Flask(__name__)

# 所有图片文件放在根目录的images文件夹下
UPLOAD_PATH = os.path.join(os.path.dirname(__file__), 'images')


@app.route('/upload/', methods=['GET', 'POST'])
def upload():
    if request.method == 'GET':
        return render_template('upload.html')
    else:
        # 结合表单request中的多种表单元素
        form = UploadFileForm(CombinedMultiDict([request.form, request.files]))
        if form.validate():
            # 根据html中对应标签的name属性获取对应上传的数据
            # request.form相当于一个字典
            # desc = request.form.get('desc')
            desc = form.desc.data
            print(desc)
            # 获取文件需要从request.files中获取
            # avatar = request.files.get('avatar')
            avatar = form.avatar.data
            # 为了安全起见，需要将文件名使用特殊方式（secure_filename函数）过滤处理一下
            # secure_filename对中文支持不是很好，可以对文件名进行转换，但是仍然推荐使用这个函数来进行处理一下
            filename = secure_filename(avatar.filename)
            # 返回的文件对象可以直接通过它的save方法传入路径保存，路径不能是相对路径，需要是绝对路径
            avatar.save(os.path.join(UPLOAD_PATH, filename))
            return '文件上传成功！'
        else:
            print(form.errors)
            return '文件上传失败！'


@app.route('/images/<filename>/')
def get_image(filename):
    # 获取文件返回到浏览器中，使用send_from_directory，第一个参数是文件目录，第二个参数是文件名
    return send_from_directory(UPLOAD_PATH, filename)


if __name__ == '__main__':
    app.run(debug=True)
```



