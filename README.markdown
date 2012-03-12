# module_info #

This is a Drupal module to simplify the process of developing custom hooks.
It handles the boiler-plate of calling all instances of an _info hook,
collating the results, merging in default values, sorting by weight, 
and calling alter hooks.

In addition, each info definition can register callback functions for various
purposes, and use this module to find and invoke them.

## Use ##

Here is how to declare a new info hook.

    function mymodule_status_info($key = NULL) {
      return module_info_invoke_all('mymodule_status_info', array(
        'title' => '',
      ), $key);
    }
  
Here is an implementation of this info hook.
  
    function mymodule_mymodule_status_info() {
      return array(
        'active' => array(
          'title' => t('Active'),
        ),
        'pending' => array(
          'title' => t('Pending'),
          'callbacks' => array(
            'apply' => 'mymodule_status_callback_pending',
          ),
        ),
        'canceled' => array(
          'title' => t('Canceled'),
          'base' => 'mymodule_callbacks',
        ),
      );
    }
  
Properties <code>module</code>, <code>key</code> and <code>weight</code>
will automatically be added to each definition, if they are not set explicitly.

The results will then be sent through <code>hook_mymodule_status_info_alter</code>
and will finally be sorted by weight before being returned.

Here is how to get an options list for these properties, mapping keys to each title property.

    function mymodule_status_options() {
      return module_info_options('mymodule_status_info', 'title');
    }
  

In addition, each definition can be tied to callback functions for specific purposes.

*   If a definition has a single 'callback' function, it will be used.
*   If a definition has a 'callbacks' property with the callback function name
    as a sup-property, its value will be used as the callback.
*   If the definition has a 'base' property, it will be prepended to the callback function name.
*   Otherwise, the module name will be prepended to the callback function name.

Of these, the first callback function that exists will be returned.

    function mymodule_status_apply($data, $status) {
      $info = mymodule_status_info($status);
      if ($info) {
        $callback = module_info_get_callback($info, 'apply');
        if ($callback) {
          return $callback($data);
        }
      }
      return FALSE;
    }
  
Here are the callback functions that will be invoked for each status,
according to the above definition.

    /**
     * The apply callback for the active status.
     */
    function mymodule_active($data) {}
    
    /**
     * The apply callback for the pending status.
     */
    function mymodule_status_callback_pending($data) {}
    
    /**
     * The apply callback for the canceled status.
     */
    function mymodule_callbacks_apply($data) {}
    
## License

Copyright New Signature 2010 - 2011

This program is free software: you can redistribute it and/or modify it under the terms of the 
GNU General Public License as published by the Free Software Foundation, either version 3 of the 
License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; 
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  
See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  
If not, see <http://www.gnu.org/licenses/>.

You can contact New Signature by electronic mail at labs@newsignature.com 
or- by U.S. Postal Service at 1100 H St. NW, Suite 940, Washington, DC 20005.