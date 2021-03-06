.. _virtualenv:

==================
Python virtualenv
==================

pip是Python包管理器，用于安装和更新软件包，需要确保系统已经安装了最新版本的pip。

使用pip可以安装Python虚拟环境管理器：对于Python 3使用venv，对于Python 2使用virtualenv。

.. note::

   在 :ref:`django_env` 同样也使用virtualenv来构建Django开发环境

Python 2 virtualenv
====================

CentOS 7通过Yum安装（EPEL源）
------------------------------

- 安装::

   yum -y update
   yum -y install python-pip

   pip2 install virtualenv
   virtualenv venv2

CentOS 8通过dnf安装python 2virtualenv
----------------------------------------

.. note::

   CentOS/RHEL 8默认使用Python 3，请参考 :ref:`python_in_rhel8`

- 安装python 2 (默认CentOS 8只安装python 3)::

   dnf install python2

- 安装python2的virtualenv::

   cd ~
   python2 -m virtualenv venv2 

- 激活virtualenv::

   source venv2/bin/active
   # 检查版本
   python --version
   # 离开virtualenv沙箱
   deactive

Python 3 venv
====================

- :ref:`python_in_rhel8` 默认安装Python 3，所以构建虚拟沙箱环境非常简单::

   cd ~
   python3 -m venv venv3

- 激活::

   source venv3/bin/active

参考
=====

- `How to Install Pip on CentOS 7 <https://www.liquidweb.com/kb/how-to-install-pip-on-centos-7/>`_
- `Installing packages using pip and virtual environments <https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/>`_
