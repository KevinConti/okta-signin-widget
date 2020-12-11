/*!
 * Copyright (c) 2018, Okta, Inc. and/or its affiliates. All rights reserved.
 * The Okta software accompanied by this notice is provided pursuant to the Apache License, Version 2.0 (the "License.")
 *
 * You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0.
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
 * WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *
 * See the License for the specific language governing permissions and limitations under the License.
 */

<template>
  <div class="container">
    <div id="login-background" class="login-bg-image"></div>
    <div class="login">
      <div id="okta-signin-container"></div>
    </div>
    <p>Note: Vue does not re-render components when url arguments change. If flipping clients, make sure to hard-refresh</p>
  </div>
</template>

<script>
import OktaSignIn from '@okta/okta-signin-widget'
import '@okta/okta-signin-widget/dist/css/okta-sign-in.min.css'

import sampleConfig from '../config'



function MapClientToBranding(client) {
  // This function can obviously be much fancier in a non-PoC
  let dict = {
    'inhere' : {
      logoFileName: 'InhereLogo.svg',
      name: 'Inhere',
      image: 'InhereBackground.jpg'
    },
    'softpro': {
      logoFileName: 'SoftproLogo.png',
      name: 'Softpro',
      image: null
    },
    'default': {
      logoFileName: 'logo.png',
      name: 'Okta',
      image: null
    }
  };

  return BuildBrand(dict[client && client.toLowerCase()] || dict['default']);

  // Convenience function to map to the correct object structure
  function BuildBrand({logoFileName, name, image}) {
    return {
      branding: {
        logo: require(`@/assets/${logoFileName}`),
        i18n: {
          en: {
            'primaryauth.title': `Sign in to ${name}`
          }
        }
      },
      backgroundImage: image
    }
  }
}


export default {
  name: 'Login',
  mounted: function () {
    this.$nextTick(function () {
      // Dynamically get branding and background image from the parameters
      let params = new URLSearchParams(window.location.search);
      let {branding, backgroundImage} = MapClientToBranding(params.get('client'));

      // Set background image
      if (backgroundImage) {
        let backgroundUrl = `url('${require(`@/assets/${backgroundImage}`)}')`
        document.getElementById('login-background').style.backgroundImage = backgroundUrl;
      }

      this.widget = new OktaSignIn({
        ...branding,
        /**
         * Note: when using the Sign-In Widget for an OIDC flow, it still
         * needs to be configured with the base URL for your Okta Org. Here
         * we derive it from the given issuer for convenience.
         */
        baseUrl: sampleConfig.oidc.issuer.split('/oauth2')[0],
        clientId: sampleConfig.oidc.clientId,
        redirectUri: sampleConfig.oidc.redirectUri,
        // eslint-disable-next-line no-undef
        authParams: {
          pkce: true,
          issuer: sampleConfig.oidc.issuer,
          display: 'page',
          scopes: sampleConfig.oidc.scopes
        }
      })

      this.widget.renderEl(
        { el: '#okta-signin-container' },
        () => {
          /**
           * In this flow, the success handler will not be called because we redirect
           * to the Okta org for the authentication workflow.
           */
        },
        (err) => {
          throw err
        }
      )
    })
  },
  destroyed () {
    // Remove the widget from the DOM on path change
    this.widget.remove()
  }
}
</script>
