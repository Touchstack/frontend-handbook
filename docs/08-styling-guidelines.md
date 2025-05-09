# Styling Guidelines

This guide outlines our approach to styling in Next.js applications, focusing on Tailwind CSS, shadcn/ui components, and Framer Motion for animations.

## Tailwind CSS

We use Tailwind CSS as our primary styling solution for its utility-first approach and flexibility.

### Setup

Ensure Tailwind is properly configured in your Next.js project:

```bash
# Install Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Configure your `tailwind.config.js`:

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        // Your custom color palette
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        // ... other colors
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        // Your custom animations
      },
      animation: {
        // Your custom animations
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

Include Tailwind directives in your CSS file:

```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;

    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;

    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;

    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

## shadcn/ui Components

We use shadcn/ui as our component library, which is built on top of Tailwind CSS and Radix UI.

### Installation

```bash
# Install CLI
npm install -D shadcn-ui

# Initialize shadcn/ui
npx shadcn-ui@latest init
```

### Using Components

Install components as needed:

```bash
# Example: Install button component
npx shadcn-ui@latest add button

# Example: Install form components
npx shadcn-ui@latest add form
```

### Component Usage

```typescript
// components/user-profile.tsx
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export function UserProfile({ user }) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{user.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p className="text-muted-foreground">{user.email}</p>
        <Button className="mt-4">Edit Profile</Button>
      </CardContent>
    </Card>
  );
}
```

## Framer Motion

We use Framer Motion for animations in our applications to create smooth, professional interactions.

### Installation

```bash
npm install framer-motion
```

### Basic Usage

```tsx
// components/animated-card.tsx
"use client";

import { motion } from "framer-motion";
import { Card, CardContent } from "@/components/ui/card";

export function AnimatedCard({ children }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 20 }}
      transition={{ duration: 0.3 }}
    >
      <Card>
        <CardContent>{children}</CardContent>
      </Card>
    </motion.div>
  );
}
```

### Common Animation Patterns

#### 1. Fade In

```tsx
const fadeIn = {
  initial: { opacity: 0 },
  animate: { opacity: 1 },
  exit: { opacity: 0 },
  transition: { duration: 0.3 }
};

function FadeInComponent() {
  return (
    <motion.div
      initial="initial"
      animate="animate"
      exit="exit"
      variants={fadeIn}
    >
      Content
    </motion.div>
  );
}
```

#### 2. Staggered List Animation

```tsx
// components/animated-list.tsx
"use client";

import { motion } from "framer-motion";

const container = {
  hidden: { opacity: 0 },
  show: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1
    }
  }
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 }
};

export function AnimatedList({ items }) {
  return (
    <motion.ul
      variants={container}
      initial="hidden"
      animate="show"
      className="space-y-3"
    >
      {items.map((item, index) => (
        <motion.li key={index} variants={item} className="p-3 bg-card rounded-md">
          {item.content}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

#### 3. Page Transitions

```tsx
// app/components/page-transition.tsx
"use client";

import { motion } from "framer-motion";

const variants = {
  hidden: { opacity: 0, x: -200, y: 0 },
  enter: { opacity: 1, x: 0, y: 0 },
  exit: { opacity: 0, x: 0, y: -100 },
};

export function PageTransition({ children }: { children: React.ReactNode }) {
  return (
    <motion.main
      variants={variants}
      initial="hidden"
      animate="enter"
      exit="exit"
      transition={{ type: "linear" }}
      className="w-full"
    >
      {children}
    </motion.main>
  );
}
```

### Combining with shadcn/ui

```tsx
// components/animated-dialog.tsx
"use client";

import { useState } from "react";
import { motion, AnimatePresence } from "framer-motion";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";

export function AnimatedDialog() {
  const [open, setOpen] = useState(false);
  
  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>Open Dialog</Button>
      </DialogTrigger>
      <AnimatePresence>
        {open && (
          <DialogContent forceMount asChild>
            <motion.div
              initial={{ opacity: 0, scale: 0.95 }}
              animate={{ opacity: 1, scale: 1 }}
              exit={{ opacity: 0, scale: 0.95 }}
              transition={{ duration: 0.2 }}
            >
              <DialogHeader>
                <DialogTitle>Animated Dialog</DialogTitle>
                <DialogDescription>
                  This dialog uses Framer Motion for smooth animations.
                </DialogDescription>
              </DialogHeader>
              <div className="p-4">Dialog content here</div>
            </motion.div>
          </DialogContent>
        )}
      </AnimatePresence>
    </Dialog>
  );
}
```

### Animation Best Practices

1. **Performance**: Keep animations simple and performant by:
   - Animating only CSS properties that trigger compositing (`opacity`, `transform`)
   - Using `will-change` sparingly
   - Avoiding animating large components

2. **Accessibility**:
   - Add `prefers-reduced-motion` support in your animations
   - Use the `useReducedMotion` hook from Framer Motion

```tsx
// components/motion-safe.tsx
"use client";

import { useReducedMotion } from "framer-motion";
import { motion } from "framer-motion";

export function MotionSafeAnimation({ children }) {
  const prefersReducedMotion = useReducedMotion();
  
  const variants = {
    hidden: { opacity: 0, y: prefersReducedMotion ? 0 : 20 },
    visible: { opacity: 1, y: 0 },
  };
  
  return (
    <motion.div
      variants={variants}
      initial="hidden"
      animate="visible"
      transition={{ duration: prefersReducedMotion ? 0 : 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

3. **Maintaining Visual Hierarchy**:
   - Use more subtle animations for less important elements
   - Reserve more dramatic animations for primary actions
   - Keep animations consistent throughout the application

4. **Animation Guidelines**:
   - Duration: 0.2s to 0.4s for most UI animations
   - Easing: Use `ease-out` for entrances, `ease-in` for exits
   - Delay: Keep delays under 0.2s to avoid feeling sluggish

## Naming Conventions

We follow these conventions for component and file naming:

- **Component Names**: Use PascalCase (e.g., `UserProfile`, `DataTable`)
- **File and Directory Names**: Use kebab-case (e.g., `user-profile.tsx`, `data-table.tsx`)
- **CSS Class Names**: Use kebab-case for custom classes outside of Tailwind utilities

Example:

```
components/
├── ui/                # shadcn components
│   ├── button.tsx
│   └── card.tsx
├── layout/
│   ├── site-header.tsx
│   └── main-nav.tsx
├── animations/        # Animation components
│   ├── fade-in.tsx
│   └── slide-in.tsx
└── features/
    └── users/
        ├── user-profile.tsx
        └── user-settings.tsx
```

## Styling Best Practices

1. **Use Tailwind Classes Directly**

```tsx
// Prefer this
<div className="flex items-center space-x-4 rounded-md border p-4">

// Over custom CSS
<div className="container">
```

2. **Create Component Variations with cn Utility**

```tsx
import { cn } from "@/lib/utils";

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "default" | "outline" | "ghost";
}

export function Button({ 
  className, 
  variant = "default", 
  ...props 
}: ButtonProps) {
  return (
    <button
      className={cn(
        "rounded-md px-4 py-2 font-medium",
        {
          "bg-primary text-primary-foreground": variant === "default",
          "border border-input bg-transparent": variant === "outline",
          "bg-transparent hover:bg-accent": variant === "ghost",
        },
        className
      )}
      {...props}
    />
  );
}
```

3. **Use @layer for Custom Utilities**

For repeated patterns, extend Tailwind with custom utilities:

```css
/* In your CSS file */
@layer components {
  .card-hover {
    @apply transition-transform duration-200 hover:scale-105;
  }
}
```

4. **Use CSS Variables for Theming**

Access theme variables in component styles:

```tsx
<div 
  className="bg-primary text-primary-foreground" 
  style={{ 
    borderRadius: "var(--radius)"
  }}
>
  Themed content
</div>
```

5. **Responsive Design**

Use Tailwind's responsive prefixes:

```tsx
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
  {/* Grid items */}
</div>
```

## Dark Mode

We support dark mode using Tailwind's dark mode feature:

```tsx
// components/theme-toggle.tsx
"use client";

import { useTheme } from "@/context/theme-context";
import { Button } from "@/components/ui/button";
import { Moon, Sun } from "lucide-react";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === "dark" ? "light" : "dark")}
    >
      {theme === "dark" ? (
        <Sun className="h-5 w-5" />
      ) : (
        <Moon className="h-5 w-5" />
      )}
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
```

## Consistent Spacing

Use Tailwind's spacing scale consistently:

- `space-y-1` to `space-y-10` for vertical spacing
- `space-x-1` to `space-x-10` for horizontal spacing
- `gap-1` to `gap-10` for grid/flex gaps
- `p-1` to `p-10` for padding
- `m-1` to `m-10` for margin

## Accessibility

Ensure styles maintain accessibility:

- Use sufficient color contrast (check with browser dev tools)
- Don't rely solely on color to convey information
- Ensure focus states are visible
- Use appropriate text sizes (min 16px for body text)
- Support `prefers-reduced-motion` for animations

## Recommended Tools

- [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss) - VS Code extension
- [shadcn/ui](https://ui.shadcn.com/) - Component documentation
- [Tailwind CSS Documentation](https://tailwindcss.com/docs) - Official docs
- [Framer Motion Documentation](https://www.framer.com/motion/) - Animation library docs
- [Motion One](https://motion.dev/) - Alternative lightweight animation library
