# Radix UI

> Accessible, unstyled UI primitives for building high-quality design systems.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Headless UI component library |
| **Why** | Accessibility, customization, composability |
| **Version** | Latest (individual packages) |
| **Location** | `apps/web/src/components/ui/` |

## Installed Components

```json
// package.json dependencies
{
  "@radix-ui/react-dialog": "^1.x",
  "@radix-ui/react-dropdown-menu": "^2.x",
  "@radix-ui/react-popover": "^1.x",
  "@radix-ui/react-select": "^2.x",
  "@radix-ui/react-tabs": "^1.x",
  "@radix-ui/react-toast": "^1.x",
  "@radix-ui/react-tooltip": "^1.x",
  "@radix-ui/react-alert-dialog": "^1.x",
  "@radix-ui/react-avatar": "^1.x",
  "@radix-ui/react-checkbox": "^1.x",
  "@radix-ui/react-label": "^2.x",
  "@radix-ui/react-slot": "^1.x",
  "@radix-ui/react-switch": "^1.x"
}
```

## Component Examples

### Dialog (Modal)
```tsx
// components/ui/dialog.tsx
import * as Dialog from '@radix-ui/react-dialog';

export function MedicationDialog({ trigger, children }) {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>
        {trigger}
      </Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/50 backdrop-blur-sm" />
        <Dialog.Content className="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white rounded-xl p-6 shadow-xl max-w-md w-full">
          <Dialog.Title className="text-xl font-semibold">
            Add Medication
          </Dialog.Title>
          <Dialog.Description className="text-gray-500 mt-2">
            Enter the medication details below.
          </Dialog.Description>
          {children}
          <Dialog.Close asChild>
            <button className="absolute top-4 right-4">
              <X className="h-4 w-4" />
            </button>
          </Dialog.Close>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

### Dropdown Menu
```tsx
import * as DropdownMenu from '@radix-ui/react-dropdown-menu';

export function UserMenu({ user }) {
  return (
    <DropdownMenu.Root>
      <DropdownMenu.Trigger asChild>
        <button className="flex items-center gap-2">
          <Avatar src={user.avatarUrl} />
          <ChevronDown className="h-4 w-4" />
        </button>
      </DropdownMenu.Trigger>

      <DropdownMenu.Portal>
        <DropdownMenu.Content className="bg-white rounded-lg shadow-lg p-2 min-w-[200px]">
          <DropdownMenu.Label className="px-2 py-1 text-sm text-gray-500">
            {user.email}
          </DropdownMenu.Label>
          
          <DropdownMenu.Separator className="h-px bg-gray-200 my-1" />
          
          <DropdownMenu.Item className="px-2 py-2 rounded hover:bg-gray-100 cursor-pointer">
            <Link href="/settings">Settings</Link>
          </DropdownMenu.Item>
          
          <DropdownMenu.Item className="px-2 py-2 rounded hover:bg-gray-100 cursor-pointer text-red-600">
            Logout
          </DropdownMenu.Item>
        </DropdownMenu.Content>
      </DropdownMenu.Portal>
    </DropdownMenu.Root>
  );
}
```

### Select
```tsx
import * as Select from '@radix-ui/react-select';

export function FamilyRoleSelect({ value, onChange }) {
  return (
    <Select.Root value={value} onValueChange={onChange}>
      <Select.Trigger className="flex items-center justify-between w-full px-3 py-2 border rounded-lg">
        <Select.Value placeholder="Select role" />
        <Select.Icon>
          <ChevronDown className="h-4 w-4" />
        </Select.Icon>
      </Select.Trigger>

      <Select.Portal>
        <Select.Content className="bg-white rounded-lg shadow-lg overflow-hidden">
          <Select.Viewport className="p-1">
            <Select.Item value="ADMIN" className="px-3 py-2 hover:bg-gray-100 rounded cursor-pointer">
              <Select.ItemText>Admin</Select.ItemText>
            </Select.Item>
            <Select.Item value="CAREGIVER" className="px-3 py-2 hover:bg-gray-100 rounded cursor-pointer">
              <Select.ItemText>Caregiver</Select.ItemText>
            </Select.Item>
            <Select.Item value="VIEWER" className="px-3 py-2 hover:bg-gray-100 rounded cursor-pointer">
              <Select.ItemText>Viewer</Select.ItemText>
            </Select.Item>
          </Select.Viewport>
        </Select.Content>
      </Select.Portal>
    </Select.Root>
  );
}
```

### Tabs
```tsx
import * as Tabs from '@radix-ui/react-tabs';

export function MedicationTabs() {
  return (
    <Tabs.Root defaultValue="active">
      <Tabs.List className="flex border-b">
        <Tabs.Trigger
          value="active"
          className="px-4 py-2 border-b-2 border-transparent data-[state=active]:border-primary-500 data-[state=active]:text-primary-600"
        >
          Active
        </Tabs.Trigger>
        <Tabs.Trigger
          value="inactive"
          className="px-4 py-2 border-b-2 border-transparent data-[state=active]:border-primary-500 data-[state=active]:text-primary-600"
        >
          Inactive
        </Tabs.Trigger>
      </Tabs.List>
      
      <Tabs.Content value="active" className="pt-4">
        <ActiveMedicationsList />
      </Tabs.Content>
      <Tabs.Content value="inactive" className="pt-4">
        <InactiveMedicationsList />
      </Tabs.Content>
    </Tabs.Root>
  );
}
```

### Toast Notifications
```tsx
import * as Toast from '@radix-ui/react-toast';

export function ToastProvider({ children }) {
  const [toasts, setToasts] = useState<ToastItem[]>([]);

  return (
    <Toast.Provider swipeDirection="right">
      {children}
      
      {toasts.map((toast) => (
        <Toast.Root
          key={toast.id}
          className="bg-white rounded-lg shadow-lg p-4 flex items-center gap-3"
        >
          <Toast.Title className="font-medium">{toast.title}</Toast.Title>
          <Toast.Description className="text-gray-500">
            {toast.description}
          </Toast.Description>
          <Toast.Close className="ml-auto">
            <X className="h-4 w-4" />
          </Toast.Close>
        </Toast.Root>
      ))}
      
      <Toast.Viewport className="fixed bottom-4 right-4 flex flex-col gap-2" />
    </Toast.Provider>
  );
}
```

### Tooltip
```tsx
import * as Tooltip from '@radix-ui/react-tooltip';

export function IconButton({ icon: Icon, tooltip, onClick }) {
  return (
    <Tooltip.Provider>
      <Tooltip.Root>
        <Tooltip.Trigger asChild>
          <button onClick={onClick} className="p-2 rounded-lg hover:bg-gray-100">
            <Icon className="h-5 w-5" />
          </button>
        </Tooltip.Trigger>
        <Tooltip.Portal>
          <Tooltip.Content
            className="bg-gray-900 text-white px-3 py-1.5 rounded text-sm"
            sideOffset={5}
          >
            {tooltip}
            <Tooltip.Arrow className="fill-gray-900" />
          </Tooltip.Content>
        </Tooltip.Portal>
      </Tooltip.Root>
    </Tooltip.Provider>
  );
}
```

## Accessibility Features

Radix provides built-in accessibility:

- **Keyboard Navigation**: Arrow keys, Enter, Escape
- **Focus Management**: Proper focus trapping in modals
- **Screen Reader**: ARIA attributes automatically applied
- **WAI-ARIA**: Follows ARIA design patterns

## Styling Patterns

### Data Attributes
```css
/* Style based on component state */
[data-state="open"] {
  animation: fadeIn 200ms;
}

[data-state="closed"] {
  animation: fadeOut 200ms;
}

[data-highlighted] {
  background-color: rgb(243 244 246);
}

[data-disabled] {
  opacity: 0.5;
  pointer-events: none;
}
```

### With Tailwind
```tsx
<Dialog.Content
  className={cn(
    'fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2',
    'bg-white rounded-xl shadow-xl p-6',
    'data-[state=open]:animate-fade-in',
    'data-[state=closed]:animate-fade-out'
  )}
/>
```

## Common Patterns

### Controlled vs Uncontrolled
```tsx
// Uncontrolled (internal state)
<Dialog.Root>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {/* ... */}
</Dialog.Root>

// Controlled (external state)
const [open, setOpen] = useState(false);

<Dialog.Root open={open} onOpenChange={setOpen}>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {/* ... */}
</Dialog.Root>
```

### Portal Usage
```tsx
// Render in document.body to avoid z-index issues
<Dialog.Portal>
  <Dialog.Overlay />
  <Dialog.Content />
</Dialog.Portal>

// Custom container
<Dialog.Portal container={customContainerRef.current}>
  {/* ... */}
</Dialog.Portal>
```

## Troubleshooting

### Z-Index Issues
- Use `Portal` to render outside parent stacking context
- Set appropriate z-index on overlay and content

### Focus Issues
- Check `asChild` prop usage
- Ensure only one `Trigger` per component

### Animation Not Working
- Use `data-[state=open]` and `data-[state=closed]` selectors
- Add `forceMount` for exit animations

---

*See also: [Tailwind CSS](tailwindcss.md), [Framer Motion](framer-motion.md)*


