---
title: Material-UI_with_Typescript
tags: React Material-UI Typescript
---
Material-UI is a React library that provides a set of components that follow Google's Material Design principals. Currently, the library is under-going an overhaul that is bring more consistant and preforment components. These changes are being actively worked on and are in a beta phase. You can install the latest with `npm i material-ui@next`. It wasn't until recently that typings were added the repository. Previously, you could install typings with `npm i --save-dev @types/material-ui`. These typings were unofficial, and they were maintaned by the community. With new beta version, Typescript developers were having to write their own "stop-gap" typings and use less-strict rules until breaking changes became less frequent. Today, I'll cover how I'm now styling my components in Typescript using Material-UI's exposed solution leveraging JSS.

If you've been working with the Material-UI beta, you're probably familar with their exposed solution for styling components. I'll spare us the theme set up, jump into styling presentaional components. Adapting a componet from the [CSS in JS section](https://material-ui-1dab0.firebaseapp.com/customization/css-in-js/) in the MUI docs, we have the following component:

```
import React from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';
import { withStyles } from 'material-ui/styles';
import Typography from 'material-ui/Typography';

const styles = theme => ({
  root: {
    color: 'inherit',
    textDecoration: 'inherit',
    '&:hover': {
      textDecoration: 'underline',
    },
  },
  primary: {
    color: theme.palette.primary[500],
  },
});

function MyLink(props) {
  const { children, classes, className, variant, ...other } = props;

  return (
    <a
      className={classNames(
        classes.root,
        {
          [classes.primary]: variant === 'primary',
        },
        className,
      )}
      {...other}
    >
      {children}
    </a>
  );
}
const MyLinkStyled = withStyles(styles)(MyLink);
```