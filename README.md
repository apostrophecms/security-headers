<div align="center">
  <img src="https://raw.githubusercontent.com/apostrophecms/apostrophe/main/logo.svg" alt="ApostropheCMS logo" width="80" height="80">

  <h1>Security Headers</h1>
  <p>
    <a aria-label="Apostrophe logo" href="https://v3.docs.apostrophecms.org">
      <img src="https://img.shields.io/badge/MADE%20FOR%20Apostrophe%203-000000.svg?style=for-the-badge&logo=Apostrophe&labelColor=6516dd">
    </a>
    <a aria-label="Test status" href="https://github.com/apostrophecms/apostrophe/actions">
      <img alt="GitHub Workflow Status (branch)" src="https://img.shields.io/github/workflow/status/apostrophecms/security-headers/Tests/main?label=Tests&labelColor=000000&style=for-the-badge">
    </a>
    <a aria-label="Join the community on Discord" href="http://chat.apostrophecms.org">
      <img alt="" src="https://img.shields.io/discord/517772094482677790?color=5865f2&label=Join%20the%20Discord&logo=discord&logoColor=fff&labelColor=000&style=for-the-badge&logoWidth=20">
    </a>
    <a aria-label="License" href="https://github.com/apostrophecms/module-template/blob/main/LICENSE.md">
      <img alt="" src="https://img.shields.io/static/v1?style=for-the-badge&labelColor=000000&label=License&message=MIT&color=3DA639">
    </a>
  </p>
</div>

This module sends the modern HTTP security headers that are expected by various security scanners. The default settings are strict as Apostrophe 3.x can support that, but see below for adjustments you may wish to make.

## Warning

Some third-party services, including Google Analytics, Google Fonts, YouTube and Vimeo, are included in the standard configuration. However even with these permissive settings, not all third-party services compatible with Apostrophe will be permitted out of the box. For instance, because they are used relatively rarely, no special testing has been done for Wufoo or Infogram. You should test your site and configure custom policies accordingly.

## Installation

To install the module, use the command line to run this command in an Apostrophe project's root directory:

```
npm install @apostrophecms/security-headers
```

## Usage

Configure the `@apostrophecms/security-headers` module in the `app.js` file:

```javascript
require('apostrophe')({
  shortName: 'my-project',
  modules: {
    '@apostrophecms/security-headers': {}
  }
});
```

The headers can be overridden by setting them as options to the module:

```javascript
// in app.js
modules: {
  'apostrophe-security-headers': {
    'X-Frame-Options': 'DENY'
  }
}
```

You can also disable a header entirely by setting the option to `false`.

The `Content-Security-Policy` header is more complex. The default response for it is the result of merging together options for individual use cases as shown below. However you may also simply set a string for it to override all of that.

### Default behavior

Here are the headers that are sent by default, with their default values:

```javascript
  // 1 year. Do not include subdomains as they could be unrelated sites
  'Strict-Transport-Security': 'max-age=31536000',
  // You may also set to DENY, however future Apostrophe modules may use
  // iframes to present previews etc.
  'X-Frame-Options': 'SAMEORIGIN',
  // If you have issues with broken images etc., make sure content type
  // configuration is correct for your production server
  'X-Content-Type-Options': 'nosniff',
  // Very new. Used to entirely disable browser features like geolocation per host.
  // Since we don't know what your site may need, we don't try to set this
  // header by default (false means "don't send the header")
  'Permissions-Policy': false,
  // Don't send a "Referer" (sp) header unless the new URL shares the same
  // origin. You can set this to `false` if you prefer cross-origin "Referer"
  // headers be sent. Apostrophe does not rely on them
  'Referrer-Policy': 'same-origin',
  // `true` means it should be computed according to the rules below.
  // You may also pass your own string, or `false` to not send this header.
  // The `policies` option and all of its sub-options are ignored unless
  // `Content-Security-Policy` is `true`.
  'Content-Security-Policy': true,

  // You do not have to copy and paste this entire example.
  // The sub-options you specify for `policies` are intelligently merged
  // with the defaults you see below. Any sub-option you specify explicitly at
  // project level overrides all of the default settings shown below for that
  // sub-option; you may set one to `false` to completely
  // disable it. You may also introduce entirely new sub-options,
  // which will also be honored.
  //
  // Policies of the same type from different sub-options are merged, with
  // the largest set of keywords and hosts enabled. This is done because
  // browsers do not support more than one style-src policy, for example, but
  // do support specifying several hosts.
  //
  // Note the HOSTS wildcard which matches all expected hosts including CDN hosts
  // and workflow hostnames.

  policies: {
    general: {
      'default-src': `HOSTS`,
      // Because it is necessary for some of the output of tiptap
      'style-src': "HOSTS 'unsafe-inline'",
      'script-src': `HOSTS`,
      'font-src': `HOSTS`,
      'frame-src': `'self'`
    },

    // Set this sub-option to false if you wish to forbid google fonts
    googleFonts: {
      'style-src': 'fonts.googleapis.com',
      'font-src': 'fonts.gstatic.com'
    },

    // Set this sub-option to false if you do not use the video widget
    oembed: {
      'frame-src': '*.youtube.com *.vimeo.com'
    },

    // Set this sub-option to false if you do not wish to permit Google Analytics and
    // Google Adsense
    analytics: {
      'default-src': '*.google-analytics.com *.doubleclick.net',
      // Note that use of google tag manager by definition brings in scripts from
      // more third party sites and you will need to add policies for them
      'script-src': '*.google-analytics.com *.doubleclick.net *.googletagmanager.com',
    }  
  }
```

#### Inline style attributes are still allowed

Note that `style-src` is set by default to permit inline style attributes. This is currently necessary
because the output of the tiptap rich text editor used in Apostrophe involves inline
styles in some cases.

#### Inline script tags are **not** allowed

Inline script tags are **not** allowed by default, as this is one of the primary benefits of using
`Content-Security-Policy`. If you do choose to output a script tag, you may do so if you use
the "nonce" provided by this module, like this:

```
<script nonce="{{ nonce }}">
  // inline script code here
</script>
```

The nonce mechanism ensures that the script was the intention of the developer and is not an XSS attack. However please note that you will lose the security benefits of this if you output other user-entered data inside the script tag without properly escaping it, for instance using the `| json` nunjucks filter.

### Custom policies

You may add any number of custom policies. Any sub-option nested in your
`policies` option is treated just like the standard cases above and merged into
the final `Content-Security-Policy` header.

### Disabling standard policies

You may set any of the standard policy sub-options above to `false` to disable them.

### Hosts wildcard

Note that the `HOSTS` wildcard is automaticalably replaced with a list of hosts including any `baseUrl` host, localized hostnames for specific locales, CDN hosts from your uploadfs configuration, and `self`. Use of this wildcard is recommended as Apostrophe pushes assets to Amazon S3, CDNs, etc. when configured to do so, including scripts and stylesheets.

> You may override the normal list of hosts for `HOSTS` by setting the `legitimateHosts` option to an array of strings. You could also extend the `legitimateHosts` method of this module at project level.
