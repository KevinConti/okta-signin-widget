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
  <div class="login">
    <div id="okta-signin-container"></div>
    <div style="text-align: center">
        <em><p>Note: This is done using the 'afterRender' event, listening for the controller, and using javascript to find the button and click it. A little hacky, but could work with some effort spent on making this logic resilient.</p></em>
        <em><p>Note 2: This relies on the <code>querySelectorAll</code> js function, which will not be supported on IE 7 (8+ is fine)</p></em>
    </div>
  </div>
</template>

<script>
import OktaSignIn from '@okta/okta-signin-widget'
import '@okta/okta-signin-widget/dist/css/okta-sign-in.min.css'

import sampleConfig from '../config'

var config = {
    baseUrl: sampleConfig.oidc.issuer.split('/oauth2')[0],
        clientId: sampleConfig.oidc.clientId,
        redirectUri: sampleConfig.oidc.redirectUri,
        // eslint-disable-next-line no-undef
        logo: require('@/assets/logo.png'),
        i18n: {
          en: {
            'primaryauth.title': 'Sign in to Vue & Company',
          }
        },
        authParams: {
          pkce: true,
          issuer: sampleConfig.oidc.issuer,
          display: 'page',
          scopes: sampleConfig.oidc.scopes
        }
}

export default {
  name: 'LoginWithAutoClick',
  mounted: function () {
    this.$nextTick(function () {
      this.widget = new OktaSignIn(config)
      
      // Auto-click logic
      this.widget.on('afterRender', function clickSendCode(context){
          if (context.controller === 'mfa-verify') {
              let button = document.querySelectorAll('[data-se="sms-send-code"]')[0];
              if (button) {
                  button.click();
              }
          }
      });

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
    this.widget.off('afterRender')
    this.widget.remove()
  }
}
</script>
