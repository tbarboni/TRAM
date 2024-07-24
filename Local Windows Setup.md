This is a slightly modified set of steps required to get TRAM to run on a windows machine. These were the step it took for me to get it running locally. This guide will assume you have nothing installed, so some steps may be redundant for some users. If you'd like to see the original guide follow this link: https://github.com/center-for-threat-informed-defense/tram/wiki/Developers.

1. First you need to make sure you have the necessary applications and packages installed. For windows this means using the **winget** tool. You then need to install **make**(https://gnuwin32.sourceforge.net/packages/make.htm). 
2. Next run the following commands in your terminal
```shell
winget install shellcheck
winget install shfmt
winget install python
winget install git.git
```
3. Then you need to clone the repository. You can do this by running the following command
```shell
git clone https://github.com/center-for-threat-informed-defense/tram.git
```
4. Then locate and change directories to the one you just cloned. If everything was done right, the command `cd tram\` should do the trick.
5. Next you need to create and activate your virtual environment using venv in Python. https://docs.python.org/3/library/venv.html (This documentation helped me)
```shell
:: This creates the venv in you current directory
python -m venv .\venv 
:: This acitvates the venv
venv\Scripts\activate.bat
```
6. Install Python package requirements
```shell
pip install -r requirements/requirements.txt
pip install -r requirements/test-requirements.txt
```
7. Install pre commit hooks using the `pre-commit install` command
8. Set up the application database
```shell
tram makemigrations tram
tram migrate
```
9. Download wordnet via NTLK in Python
```python
python # open python in your shell
import ntlk
ntlk.download('wordnet')
(ctrl+z) # to return back to cmd
```
10. Run the Machine learning training.
```shell
tram attackdata load
tram pipeline load-training-data
tram pipeline train --model nb
tram pipeline train --model logreg
tram pipeline train --model nn_cls
```
11. Download the pre-trained tokenizer 
```python
python # open python in your shell again
import os; import transformers; mdl = transformers.AutoTokenizer.from_pretrained('allenai/scibert_scivocab_uncased'); mdl.save_pretrained('data/ml-models/priv-allenai-scibert-scivocab-uncased')
(ctrl+z) # to return back to cmd
```
12. Download the BERT models
```shell
curl -L "https://ctidtram.blob.core.windows.net/tram-models/single-label-202308303/config.json" -o data/ml-models/bert_model/config.json
curl -L "https://ctidtram.blob.core.windows.net/tram-models/single-label-202308303/pytorch_model.bin" -o data/ml-models/bert_model/pytorch_model.bin
```
13. Create a superuser (web login)
```shell
tram createsuperuser
```
14. Navigate to the \tram\src\tram\settings.py file and modify the if else statement on line 56 to always be True. (Their probably is a better way to do this, but it still works). This will ensure that debug mode is always on when running TRAM. It should look like this:
```python
# SECURITY WARNING: don't run with debug turned on in production!
_django_debug_env = os.environ.get("DJANGO_DEBUG")
if _django_debug_env is not None:
    DEBUG = _django_debug_env.lower() in ["true", "1", "t", "yes", "y"]
else:
    DEBUG = True
```
15. Start the server using the command `tram runserver`
16. Navigate to http://127.0.0.1:8000 and login using the super user account you made.
