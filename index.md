<style>
@font-face { font-family: ff; src: url('fo.ttf'); } 
</style>
<div dir="rtl" style="font-family: 'ff'"> 
<p>
از پایتون 2.5 <code>with statement</code> معرفی شد. کاربرد اصلی این عبارت موقعی است که دو تکه کد ثابت داریم که بین آن‌ها کدهای مختلفی ممکن است اجرا شود ولی دو تکه کد ابتدایی و پایانی ثابت هستند. بهترین مثال برای توضیح، خواندن فایل است. به صورت عمومی در زبان‌های برنامه نویسی ابتدا فایل باز میشود، در آن تغییراتی ایجاد میشود و سپس فایل بسته میشود. در پایتون نیز به همین صورت است. باز کردن و بستن فایل دو عمل تکراری و ثابت این فرایند هستند که با استفاده از <code>with</code> میتوان آن‌را سریعتر و کوتاهتر نوشت:
</p>
<div dir="ltr">
<pre>
<code>
with open(‘file.txt’, ‘w’) as f: 
    f.write(‘Hello!’)
</code>
</pre>
</div>
<p>
در تکه کد بالا ابتدا فایل <code>file.txt</code> برای نوشتن باز میشود (با نام <code>f</code>) سپس در آن <code>Hello!</code> نوشته میشود. بستن فایل به صورت خودکار توسط <code>with statement</code> انجام میشود. اگرچه در <code>CPython</code> مکانیزم <code>garbage collection</code> به صورت شمارش رفرنس است و به احتمال زیاد تا مدت بسیار کوتاهی پس از اجرای برنامه حتی اگر فایل را نبسته باشیم (یا از <code>with</code> استفاده نکرده باشیم) خود سیستم فایل را میبندد اما این موضوع کاملا به پیاده سازی مفسر بستگی دارد و کدی که با <code>CPython</code>  مشکلی ندارد ممکن است در <code>iPython</code> پیاده سازی‌ها به مشکل برخورد کند (به اصطلاح کد <code>Pythonic</code> نیست).
</p>

<p>
اما <code>with</code> چگونه کار میکند؟ ابتدا حالت کلی را معرفی میکنیم. در عبارت 
</p>
<div dir="ltr">
<pre>
<code>
with object (as target):
</code>
</pre>
</div>

<p>
اگر در کلاس مربوط به <code>object</code> دو تابع <code>__enter__</code> و <code>__exit__</code> پیاده سازی شده باشند، این خط کد <code>valid</code> است. ابتدا <code>object</code> در یک متغیر <code>hidden</code> برای برنامه نویس ذخیره میشود (در صورتی که عبارت دل‌بخواهی <code>as target</code> آورده شده باشد، در این متغیر نیز یک رفرنس ذخیره میشود) سپس تابع <code>__enter__</code> روی <code>target</code> یا متغیر پنهانی فراخوانی میشود. سپس هر آنچه در داخل <code>block</code>  مربوطه باشد اجرا میشود (در مورد <code>exception</code> جلوتر توضیح خواهم داد) و در نهایت تابع <code>__exit__</code> روی <code>target</code> یا متغیر پنهانی اجرا میشود.
</p>

<p>
حال کد ابتدایی برای نوشتن در فایل را درنظر بگیرید، در این کد یک <code>object</code> از کلاس <code>TextIOWrapper</code> بازگردانده میشود و در <code>f</code> یک رفرنس به آن ذخیره میشود. سپس روی این <code>object</code> تابع <code>__enter__</code> صدا زده میشود. پس از اجرای کد داخل بلاک <code>with</code> ، روی این <code>object</code> تابع <code>__exit__</code> صدا زده میشود که با مشاهده سورس مربوط به آن متوجه میشویم که روی فایل، تابع <code>close</code>  صدا زده شده و این باعث میشود نیازی به انجام این کار به صورت <code>explicit</code> نباشد.
</p>

<p>
اما در صورتی که در بلاک <code>with</code> اکسپشن رخ دهد چه اتفاقی میوفتد؟ ابتدا <code>with statement</code> این اکسپشن را <code>catch</code>  میکند، سپس بلافاصله تابع <code>__exit__</code> روی <code>object</code> مربوطه صدا زده میشود. در این تابع میتوان از روی <code>return value</code> یا هر روش دیگری متوجه رخ دادن اکسپشن شد و آن‌را به درستی هندل کرد. در انتها کد یک کلاس دلخواه را برای اجرای درست <code>with</code> می‌آوریم.
</p>

<div dir="ltr">
<pre>
<code>
class Saved():
    def __init__(self, val):
        self.val = val
    def __enter__(self):
        self.val.save()
        return self.val
    def __exit__(self, type, value, traceback):  # if any exception happens, type, value and traceback send to this function.
        self.val.restore()
        
with Saved(val):       # based on __enter__ method, val is returned here.
   val.do_something()  # after this block executes __exit__ is called on val.
</code>
</pre>
</div>

</div>
