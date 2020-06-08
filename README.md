# OBS Studio Python Scripting Cheetsheet
Python3.6 , OBS Studio 25.0.8 
# Table of content 
- [Using classes](#using-classes)
- [with statement](#with-statement)
- [Passing arguments to callbacks](#passing-arguments-to-callbacks)
- [UI](#ui)
- [Additional input](#additional-input)
- [obs_data](#obs_data)
- [Timing (sequential primitives) ](#timing-(sequential-primitives))
- [Hotkey](#hotkey)
- [Contribute](#contribute)

## Using classes 
```python
class Example:
    def __init__(self,source_name=None):
        self.source_name = source_name

    def update_text(self):
        source = obs.obs_get_source_by_name(self.source_name)
        if source is not None:
            data = str(next(datacycle))
            settings = obs.obs_data_create()
            obs.obs_data_set_string(settings, "text", data)
            obs.obs_source_update(source, settings)
            obs.obs_data_release(settings)
            obs.obs_source_release(source)
```
[Full example](example_class.py)

## with statement 
Automatically release .

```python
@contextmanager
def source_auto_release(source_name):
    source = obs.obs_get_source_by_name(source_name)
    try:
        yield source 
    finally:
        obs.obs_source_release(source)
...
# usage
with source_auto_release(self.source_name) as source:
    if source is not None:
        data = str(next(datacycle))
        with data_auto_release() as settings:
            obs.obs_data_set_string(settings, "text", data)
            obs.obs_source_update(source, settings)
```
[Full example](with_stmt.py)  
See also :   
https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager
## Passing arguments to callbacks
```python
from functools import partial
...
flag = obs.obs_data_get_bool(settings,"_obs_bool")
eg.update_text = partial(eg.update_text,flag_func=flag)
...
```
[Full example](callback_partial.py)
## UI
|code   | result  |
| ---   | ---     |
|`obs.obs_properties_add_button(props, "button1", "Refresh1:",callback)` | ![img](button.png) |
|`obs.obs_properties_add_bool(props,"_bool","_bool:")` | ![img](bool.png) |
|`obs.obs_properties_add_int(props,"_int","_int:",1,100,1)` | ![img](int.png) |
|`obs.obs_properties_add_int_slider(props,"_slider","_slider:",1,100,1) ` | ![img](slider.png) |
|`obs.obs_properties_add_text(props, "_text", "_text:", obs.OBS_TEXT_DEFAULT) ` | ![img](text.png) |
|`obs.obs_properties_add_color(props,"_color","_color:") ` | ![img](color.png) |
|`obs.obs_properties_add_font(props,"_font","_font:")  `|  ![img](font.png) |

See also :   
https://obsproject.com/docs/reference-properties.html#property-object-functions

## Additional input 
```python
def callback(props, prop, settings):
    _number = obs.obs_data_get_int(settings, "_int")
    _text_value = obs.obs_data_get_string(settings, "_text")
    text_property = obs.obs_properties_get(props, "_text")
    if _number > 50:
        eg.data = _text_value + str(_number)
        obs.obs_property_set_visible(text_property, True)
        return True
    else:
        eg.data = ""
        obs.obs_property_set_visible(text_property, False)
        return True
...

def script_properties():  # ui

    ...
    number = obs.obs_properties_add_int(props, "_int", "Number", 1, 100, 1)
    text_value = obs.obs_properties_add_text(
        props, "_text", "Additional input:", obs.OBS_TEXT_DEFAULT
    )
    obs.obs_property_set_visible(text_value, False)
    obs.obs_property_set_modified_callback(number, callback)
    ...
```
[Full example](modification_prop.py)  
See also :  
https://obsproject.com/docs/reference-properties.html#property-modification-functions

## obs_data
```python
    ...
    data = vars(obs)
    with open('export1.txt','w') as f:
        pprint(data,stream=f,width=100)
    ...

```
[Full example](export_vars.py)  
[Generated export1.txt](export1.txt) contains all variables available in `obspython`  

Overall , properties share similar structure , in Python, Lua, C.
[Example C](https://github.com/obsproject/obs-studio/blob/05c9ddd2293a17717a1bb4189406dfdad79a93e1/plugins/oss-audio/oss-input.c#L626)


# Timing (sequential primitives)

```python
def script_update(settings):
    eg.source_name = obs.obs_data_get_string(settings, "source")
    obs.timer_remove(eg.update_text)
    if eg.source_name != "":
        obs.timer_add(eg.update_text, 1 * 1000)
```
[Full example](example_class.py)  
Note: each time script updated it's removed first  
See also :   
https://obsproject.com/docs/scripting.html#script-timers  

# Hotkey
```python
class Example:
    def __init__(self, source_name=None):
        self.source_name = source_name
        self.hotkey_id_htk = obs.OBS_INVALID_HOTKEY_ID
    ...
def script_save(settings):
    hotkey_save_array_htk = obs.obs_hotkey_save(eg.hotkey_id_htk)
    obs.obs_data_set_array(settings, "htk_hotkey", hotkey_save_array_htk)
    obs.obs_data_array_release(hotkey_save_array_htk)
    ...
def script_load(settings):
    def callback(pressed):
        if pressed:
            return eg.update_text()

    hotkey_id_htk = obs.obs_hotkey_register_frontend(
        "htk_id", "Example hotkey", callback
    )
    hotkey_save_array_htk = obs.obs_data_get_array(settings, "htk_hotkey")
    obs.obs_hotkey_load(hotkey_id_htk, hotkey_save_array_htk)
    obs.obs_data_array_release(hotkey_save_array_htk)
    ...
```
[Full example](hotkey_exmpl.py)
# Contribute
Missing something ? Fork & PR , contributions are welcome!