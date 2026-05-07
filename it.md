[2026-05-08 01:52:49,745] ERROR in app: Exception on /api/menu/add [POST]
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/flask/app.py", line 1511, in wsgi_app
    response = self.full_dispatch_request()
  File "/usr/lib/python3/dist-packages/flask/app.py", line 919, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/usr/lib/python3/dist-packages/flask/app.py", line 917, in full_dispatch_request
    rv = self.dispatch_request()
  File "/usr/lib/python3/dist-packages/flask/app.py", line 902, in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)  # type: ignore[no-any-return]
           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^
  File "/home/admin/SmartKiosk1/kiosk_app/api_server.py", line 41, in add_product
    price = int(request.form.get('price'))
ValueError: invalid literal for int() with base 10: '10000.0'
192.168.131.152 - - [08/May/2026 01:52:49] "POST /api/menu/add HTTP/1.1" 500 -
