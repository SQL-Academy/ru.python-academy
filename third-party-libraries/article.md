# Third-Party Libraries and pip

Python's built-in libraries are just a basic set of tools, like a screwdriver and a hammer. But to create truly amazing projects, you need specialized tools. And this is where third-party libraries come to the rescue! 🛠️

## What are third-party libraries?

> Third-party libraries are Python modules that are not included in the standard library and are developed by independent developers or organizations.

Third-party libraries help:

-   Solve specific tasks in different areas
-   Save time by using ready-made solutions
-   Simplify complex operations that require deep knowledge
-   Create more powerful and functional applications

## What is pip?

> pip (Package Installer for Python) is a package management system used to install and manage software packages written in Python.

pip allows you to:

-   Install packages from the Python Package Index (PyPI) and other sources
-   Update and remove packages
-   Manage dependencies (other packages required for operation)
-   Create a list of project dependencies for subsequent installation

## Installing and using pip

Usually pip is already installed along with Python. You can check the presence and version of pip like this:

```bash
pip --version
```

Expected result:

```bash
pip 23.1.2 from /usr/local/lib/python3.11/site-packages/pip (python 3.11)
```

### Basic pip commands

#### Installing a package

```bash
pip install requests
```

Expected result:

```bash
Collecting requests
  Downloading requests-2.31.0-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.6/62.6 kB 1.2 MB/s eta 0:00:00
Installing collected packages: requests
Successfully installed requests-2.31.0
```

#### Installing a specific version

```bash
pip install requests==2.25.1
```

Expected result:

```bash
Collecting requests==2.25.1
  Downloading requests-2.25.1-py2.py3-none-any.whl (61 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 61.2/61.2 kB 1.8 MB/s eta 0:00:00
Installing collected packages: requests
Successfully installed requests-2.25.1
```

#### List of installed packages

```bash
pip list
```

Expected result:

```bash
Package    Version
---------- ---------
certifi    2023.5.7
charset-normalizer 3.1.0
idna       3.4
pip        23.1.2
requests   2.31.0
setuptools 67.8.0
urllib3    2.0.3
```

#### Removing a package

```bash
pip uninstall requests -y
```

Expected result:

```bash
Found existing installation: requests 2.31.0
Uninstalling requests-2.31.0:
  Successfully uninstalled requests-2.31.0
```

## Popular third-party libraries

Python has a huge number of third-party libraries for a wide variety of tasks. Here are some of the most popular and useful:

| Library            | Description                                       | Main application                              | Link                                                               |
| ------------------ | ------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------ |
| **Requests**       | Convenient handling of HTTP requests              | Interaction with web resources, APIs          | [Documentation](https://requests.readthedocs.io/)                  |
| **Pandas**         | Powerful tool for data analysis                   | Processing and analyzing tabular data         | [Documentation](https://pandas.pydata.org/)                        |
| **NumPy**          | Working with arrays and mathematical calculations | Scientific computing, array operations        | [Documentation](https://numpy.org/)                                |
| **Matplotlib**     | Data visualization                                | Creating plots and charts                     | [Documentation](https://matplotlib.org/)                           |
| **Seaborn**        | Statistical data visualization                    | Beautiful statistical graphics                | [Documentation](https://seaborn.pydata.org/)                       |
| **scikit-learn**   | Machine learning                                  | Building and training machine learning models | [Documentation](https://scikit-learn.org/)                         |
| **TensorFlow**     | Deep learning and neural networks                 | Complex deep learning models                  | [Documentation](https://www.tensorflow.org/)                       |
| **PyTorch**        | Deep learning with dynamic computational graphs   | Research in deep learning                     | [Documentation](https://pytorch.org/)                              |
| **Flask**          | Micro-framework for web development               | Creating web applications and APIs            | [Documentation](https://flask.palletsprojects.com/)                |
| **Django**         | Full-featured web framework                       | Large web projects                            | [Documentation](https://www.djangoproject.com/)                    |
| **Beautiful Soup** | Parsing HTML and XML                              | Extracting data from web pages                | [Documentation](https://www.crummy.com/software/BeautifulSoup/)    |
| **Pillow**         | Image processing                                  | Editing and analyzing images                  | [Documentation](https://pillow.readthedocs.io/)                    |
| **SQLAlchemy**     | ORM for working with databases                    | Interaction with SQL databases                | [Documentation](https://www.sqlalchemy.org/)                       |
| **Pygame**         | Creating games and multimedia applications        | 2D game development                           | [Documentation](https://www.pygame.org/)                           |
| **PyQt**           | Creating desktop applications                     | Graphical user interfaces (GUI)               | [Documentation](https://www.riverbankcomputing.com/software/pyqt/) |

The choice of library depends on the specific task you want to solve. The Python community is very active, and ready-made solutions already exist for most practical tasks, which can be installed via pip.

Example of installing a popular library:

```bash
# Installing pandas for data analysis
pip install pandas

# Installing Flask for web development
pip install flask
```

## Virtual environments

When working with different projects, you often need to use different versions of libraries. For this purpose, Python has virtual environments.

> A virtual environment is an isolated Python environment where you can install packages without affecting other projects or the system Python.

### Creating a virtual environment

```bash
# Creating a virtual environment
python -m venv myenv

# Activating the virtual environment
# On Windows:
myenv\Scripts\activate

# On macOS/Linux:
source myenv/bin/activate

# After activation, the environment name will appear at the beginning of the command prompt
(myenv) $
```

### Installing packages in a virtual environment

```bash
# Installing packages in the activated virtual environment
pip install pandas matplotlib
```

### Saving and installing dependencies

```bash
# Saving the list of installed packages
pip freeze > requirements.txt

# The contents of the requirements.txt file will look something like this:
# matplotlib==3.7.2
# numpy==1.25.2
# pandas==2.0.3
# ...

# Installing packages from the requirements.txt file
pip install -r requirements.txt
```

## Understanding Check

Let's check how well you've learned about third-party libraries and pip:

**Which command will correctly install pandas version 1.5.0?**
