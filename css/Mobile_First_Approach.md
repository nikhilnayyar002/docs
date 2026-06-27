# General

Mobile layouts are naturally simple—usually featuring vertical stacked grids, single-column lists, and hidden sidebars. Desktop layouts are complex, with multi-column grids etc.

It forces you to think about what content is most important on mobile devices, prioritize that content on mobile devices, and then add presentational elements—that aren’t as essential—to larger devices. From a technological viewpoint, it can also be easier to build up a page, rather than tear down. For instance, instead of having floats for the desktop, only to remove them on mobile (requiring two styles), mobile first would start with the default stacking of elements (no style required for that) and simply add one style for larger displays that adds the floats for columns.

Google indexes the mobile version. A site that loads well on phones and has a clear structure ranks better.

# Tailwind

On mobile (`< 768px`): `text-2xl`. From `768px` (`md`) up: `text-3xl` overrides. From `1024px` (`lg`) up: even larger. No manual media queries.

