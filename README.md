# pygetfacl

*Pygetfacl* retrieves Access Control List (ACL) information provided by the Linux [`getfacl`](https://manpages.ubuntu.com/manpages/trusty/man1/getfacl.1.html) command and stores that information as a Python object.



## System Requirements

* Linux distribution with the `acl` package installed.
* Python version 3.7 or higher



## Primary Use Case

*Pygetfacl* may be useful any time you want to obtain a file path's ACL information from within a Python program.



## Installation

From the command line:
```shell
$ pip install git+https://github.com/duanegoodner/pygetfacl
```



## Basic Usage

> **Note**
> If you want to test pygetfacl in Docker, you can build and run a container using files provided in the `demo_docker` directory. From the pygetfacl project root directory, run the following terminal commands to build/run the container and start a `bash` shell inside it. Pygetfacl is installed during the image build process.
>
> ```shell
> $ docker build ./demo_docker -t pygetfacl_demo
> $ docker run -it -d --rm --name="pygetfacl_demo" pygetfacl_demo
> $ docker exec -it -w /home/user_a pygetfacl_demo /bin/bash
> ```

From the command line (either in your local environment, or a Docker container), run the following command to create a test directory with some interesting ACL settings, and start the Python interpreter in interactive mode.
```shell
$ mkdir test_dir \
  && sudo useradd user_b \
  && setfacl -bn test_dir \
  && setfacl -m u:user_b:rwx test_dir \
  && chmod g+s test_dir \
  && python
```

Then, in the Python interpreter: 
```pycon
>>> from pygetfacl import ACLInfoRetriever
>>> retriever = ACLInfoRetriever("test_dir")  
>>> getfacl_result = retriever.getfacl()
```

The call to `retriever.getfacl()` returns a **GetFaclResult** object. An **ACLData** object containing ACL info for path `test_dir` can be accessed via `.acl_data` member of the **GetFaclResult** object:
```
>>> import pprint
>>> pprint.pprint(getfacl_result.acl_data)
ACLData(owning_user='user_a',
        owning_group='user_a',
        flags=-s-,
        user_permissions=rwx,
        special_users_permissions={'user_b': rwx},
        group_permissions=r-x,
        special_groups_permissions=None,
        mask=rwx,
        other_permissions=r-x,
        default_user_permissions=None,
        default_special_users_permissions=None,
        default_group_permissions=None,
        default_special_groups_permissions=None,
        default_mask=None,
        default_other_permissions=None
```



## A Few More Details



#### Permissions and flags are stored a booleans in an ACLData object

Permissions and flags in an ACLData have \__repr__ methods defined so they print appear in the usual string form when printed (e. g. `rwx` or `sst`), but the value of each bit is stored as a boolean. 

```pycon
>>> acl_data = getfacl_result.acl_data
>>> user_b_permissions = acl_data.special_users_permissions.get("user_b")
>>> print(f"r = {user_b_permissions.r}, w = {user_b_permissions.w}, x = {user_b_permissions.x}")
r = True, w = False, x = True
>>> if user_b_permissions.w:
...     print("Oh good. user_b has write access to test_dir")
... 
Oh good. user_b has write access to test_dir
```



#### Viewing raw output from Linux `getfacl` command

An **ACLInfoRetriever**'s `.get()` method returns a GetFaclResult object which has two public properties: **.raw_std_out** and **.acl_data**

The **.raw_std_out** property of a GetFaclResult object returns ACL information in the same string format provided by the Linux system getfacl:

```pycon
>>> print(getfacl_result.raw_std_out)
# file: test_dir
# owner: user_a
# group: user_a
# flags: -s-
user::rwx
user:user_b:rwx
group::r-x
mask::rwx
other::r-x
```



## Limitations

*Pygetfacl* does not offer any methods for changing ACL settings (or even "regular" permission ). For that, you may want to look at:
* [pylibacl](https://pypi.org/project/pylibacl/)
* [miracle-acl](https://pypi.org/project/miracle-acl/)
* [trigger.acl](https://pythonhosted.org/trigger/api/acl.html#module-trigger.acl)
* the standard library [`pathlib.Path.chmod()`](https://docs.python.org/3/library/pathlib.html#pathlib.Path.chmod) or [`os.chmod()`](https://docs.python.org/3/library/os.html#os.chmod) methods (for "regular" permissions only)
* calling the necessary system commands using the standard library [subprocess](https://docs.python.org/3/library/subprocess.html) module. This option usually works well for me, perhaps because my use cases tend to be simple.



## Project Status

*Pygetfacl* is a work-in-progress. Roadmap (as of March 1, 2023):

* Use current version (0.0.1) as a dependency in other projects over the next ~2 weeks.

  * During this time, no changes to project except docstring cleanup (change from Google format to reST) and improving tests.

* Based on use experience identify and implement areas for improvement. ETA for 0.0.2 release: March 14, 2023. 

  