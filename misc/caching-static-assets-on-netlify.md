<!-- 2020-08-13 netlify, hosting, jamstack -->

# Caching static assets on Netlify

By default [Netlify don’t cache files](https://www.netlify.com/blog/2017/02/23/better-living-through-caching/) which might be good for HTML but not so good for fonts or images. I could clearly see font flickering on my site with the default configuration, and it’s not the best user experience. Luckily, we can [define custom headers](https://docs.netlify.com/routing/headers/).

## Static sites

Create the **\_headers** file in your public folder (`static` for Gatsby sites):

```
# Cache fonts forever
/fonts/*
  Cache-Control: public
  Cache-Control: max-age=365000000
  Cache-Control: immutable

# Cache images for a week
/images/*
  Cache-Control: public
  Cache-Control: max-age=604800
```

Here, we’re caching font forever, and images for a week. This make the experince noticably better, and removes font flickering, and unnecessary requests to images that arn’t going to update often.

## Gatsby sites

Gatsby’s [gatsby-plugin-netlify](https://www.gatsbyjs.com/plugins/gatsby-plugin-netlify/) enables caching for Gatsby build files but it also overwrites user’s **\_headers** file. We need to use the plugins’s `headers` option.

Update Gatsby config file, **gatsby-config.js**:

```js
module.exports = {
  plugins: [
    {
      resolve: 'gatsby-plugin-netlify',
      options: {
        headers: {
          // Cache fonts forever
          '/fonts/*': [
            'Cache-Control: public',
            'Cache-Control: max-age=365000000',
            'Cache-Control: immutable'
          ],
          // Cache images for a week
          '/images/*': [
            'Cache-Control: public',
            'Cache-Control: max-age=604800'
          ]
        }
      }
    }
  ]
};
```

The format is very similar to the **\_headers** file above.
