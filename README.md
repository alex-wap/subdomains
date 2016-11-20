# Multiple Applications and Subdomains Tutorial


This tutorial teaches developers how to deploy multiple apps to the same AWS EC2 instance and how to configure subdomains. (Node and [Pylot](https://github.com/Ketul-Patel/Pylot/tree/development) apps)


## Instructions


1. First, you will need to configure at least one application for deployment. Follow the instructions below for the corresponding app type.

  * [Node Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/node_deploy.html)

  * [Pylot Deployment](https://htmlpreview.github.io/?https://github.com/alex-wap/subdomains/blob/master/pylot_deploy.html)

2. All of your applications must be on a different port.
  * Node: Typically, the port is specified in the server.js file or a settings.js file using: 
```javascript
var server = app.listen(8001);
```

  * Pylot: The port must be modified in both manage.py and wsgi.py.
  * manage.py:
```python
manager.add_command('runserver', Server(host='127.0.0.1',port='5001'))
```  
**NOTE:** port must be a string.

  * wsgi.py: 
```python
application.run(host='127.0.0.1',port=5001)
```  
**NOTE:** port must be an integer. 


3.
4.
5.
6.
7.
