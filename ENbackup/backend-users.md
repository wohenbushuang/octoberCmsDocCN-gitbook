# Backend Users & Permissions

- [Registering permissions](#permission-registration)
- [Restricting access to back-end pages](#page-access)
- [Restricting access to features](#features)

<a name="permission-registration"></a>
## Registering permissions

Plugins can register back-end user permissions by overriding the `registerPermissions()` method inside the [Plugin registration class](../plugin/registration#registration-file).  The permissions are defined as an array with keys corresponding the permission keys and values corresponding the permission descriptions. The permission keys consist of the author name, the plugin name and the feature name. Here is an example code:

    acme.blog.access_categories

The next example shows how to register back-end permission items. Permissions are defined with a permission key and description. In the back-end permission management user interface permissions are displayed as a checkbox list. Back-end controllers can use permissions defined by plugins for restricting the user access to [pages](#page-access) or [features](#features).

    public function registerPermissions()
    {
        return [
            'acme.blog.access_posts' => [
                'label' => 'Manage the blog posts',
                'tab' => 'Blog'
            ],
            'acme.blog.access_categories' => [
                'label' => 'Manage the blog categories',
                'tab' => 'Blog'
            ]
        ];
    }

<a name="page-access"></a>
## Restricting access to back-end pages

In a back-end controller class you can specify which permissions are required for access the pages provided by the controller. It's done with the `$requiredPermissions` controller's property. This property should contain an array of permission keys. If the user permissions match any permission from the list, the framework will let the user to see the controller pages.

    <?php namespace Acme\Blog\Controllers;

    use Backend\Classes\BackendController;

    class Posts extends BackendController
    {
        public $requiredPermissions = ['acme.blog.access_posts'];

You can also use the **asterisk** symbol to indicate the "all permissions" condition. In the next example the controller pages are accessible for all users who has any permissions starting with the "acme.blog." string:

    public $requiredPermissions = ['acme.blog.*'];

<a name="features"></a>
## Restricting access to features

The back-end user model has methods that allow to determine whether the user has specific permissions. You can use this feature in order to limit the functionality of the back-end user interface. The permission methods supported by the back-end user are `hasPermission()` and `hasAccess()`. The both methods take two parameters: the permission key string (or an array of key strings) and an optional parameter indicating that all permissions listed with the first parameters are required.

The `hasAccess()` method returns **true** for any permission if the user is an administrator. The `hasPermission()` method is more strict. The following example shows how to use the methods in the controller code:

    if ($this->user->hasAccess('acme.blog.*'))
        ...

    if ($this->user->hasPermission(['acme.blog.access_posts', 'acme.blog.access_categories']))
        ...

You can also use the methods in the back-end views for hiding user interface elements. The next examples demonstrates how you can hide a button on the Edit Category [back-end form](forms):

    <?php if ($this->user->hasAccess('acme.blog.delete_categories')): ?>
        <button
            type="button"
            class="oc-icon-trash-o btn-icon danger pull-right"
            data-request="onDelete"
            data-load-indicator="Deleting Category..."
            data-request-confirm="Do you really want to delete this category?">
        </button>
    <?php endif ?>
