# Odoo Custom Addons Directory

This directory contains **custom Odoo addons/modules** that will be automatically loaded by the Odoo container. The mapping is defined in [`docker-compose.yml`](../docker-compose.yml):

```yaml
volumes:
  - ./addons:/mnt/extra-addons
```

## How to Use
1. **Place your custom modules** directly in this directory (each module in its own subfolder)
2. **Restart the container** after adding new modules:
   ```bash
   docker compose restart odoo19
   ```
3. **Update module list** in Odoo:
   - Go to `Apps` → `Update Apps List`
   - Search for your new module

## Required Structure
Each addon must follow Odoo's standard module structure:
```
addons/
├── your_module_name/          # Must match module technical name
│   ├── __manifest__.py       # REQUIRED (contains module metadata)
│   ├── __init__.py           # REQUIRED (imports module components)
│   ├── models/               # Python model definitions
│   ├── views/                # XML views (forms, trees, etc.)
│   └── ...                   # Other components (static, security, etc.)
```

## Key Notes
- ✅ **Supported**: Community modules, custom developments, and modified official modules
- ❌ **Not Supported**: Direct modifications to core Odoo files (use proper module inheritance)
- 🔒 **Permissions**: Modules must be readable by UID 1000 (Odoo's default user in container)
- 🌐 **Path in Container**: All addons are available at `/mnt/extra-addons` inside the container
- ⚠️ **Critical**: The `__manifest__.py` file is **required** for Odoo to recognize your module

## Example Workflow
1. Create new module:
   ```bash
   mkdir -p addons/my_sales_customization
   ```
2. Add minimal files:
   ```python
   # addons/my_sales_customization/__manifest__.py
   {
       'name': 'My Sales Customization',
       'version': '1.0',
       'depends': ['sale'],
       'data': ['views/sale_order_views.xml'],
   }
