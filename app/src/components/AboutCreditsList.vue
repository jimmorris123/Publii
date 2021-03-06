<template>
    <dl
        class="credits-list"
        ref="content">
        <template v-for="(licenseData, index) in licensesData">
            <dt class="credits-item">
                {{ licenseData.name }}
                <a
                    class="credits-toggle"
                    @click.prevent="itemClicked($event, licenseData.id, licenseData.url)"
                    :href="licenseData.href"
                    :target="licenseData.target">
                    License
                </a>

                <a
                    v-if="licenseData.homepage"
                    :href="licenseData.homepage"
                    target="_blank">
                    Homepage
                </a>
            </dt>

            <dd :class="licenseData.cssClasses" :data-id="licenseData.id">
                <pre></pre>
            </dd>
        </template>
    </dl>
</template>

<script>
import { remote, ipcRenderer } from 'electron';

export default {
    name: 'about-credits-list',
    props: [
        'licenses'
    ],
    data: function() {
        return {
            openedID: -1
        };
    },
    computed: {
        licensesData: function() {
            let licensesData = [];
            let licensePackages = Object.keys(this.licenses).sort();
            let licenseID = 1;

            for(let licenseKey of licensePackages) {
                let licenseObject = this.parseLicense(licenseKey, licenseID);
                licensesData.push(licenseObject);
                licenseID++;
            }

            return licensesData;
        }
    },
    methods: {
        itemClicked: function(e, id, licenseUrl) {
            if(e.target.getAttribute('href') === '#') {
                if(this.openedID === id) {
                    this.openedID = -1;
                    return;
                }

                this.openedID = id;
                this.loadLicense(e.target, licenseUrl);
            }
        },
        loadLicense: function(link, licenseUrl) {
            ipcRenderer.send('app-license-load', {
                url: licenseUrl
            });

            ipcRenderer.once('app-license-loaded', function(event, licenseText) {
                link.parentNode.nextElementSibling.querySelector('pre').innerHTML = licenseText;
            });
        },
        parseLicense: function(licenseKey, licenseID) {
            let licenseHomepage = this.licenses[licenseKey].url;
            let licenseName = licenseKey.split('@')[0];
            let licenseUrl = `licenses/${licenseName}/license.txt`;
            let licenseExternal = false;
            let licenseHref = '#';
            let licenseTarget = '';

            if(!licenseHomepage) {
                licenseHomepage = this.licenses[licenseKey].repository;
            }

            if(this.licenses[licenseKey].licenseFile) {
                licenseUrl = this.licenses[licenseKey].licenseFile;
            }

            if(this.licenses[licenseKey]['helper-text']) {
                let appDirPath = remote.app.getAppPath();
                licenseUrl = 'file:///' + appDirPath + '/' + this.licenses[licenseKey].url;
                licenseHomepage = false;
                licenseExternal = true;
                licenseHref = licenseUrl;
                licenseTarget = '_blank';
            }

            return {
                id: licenseID,
                name: licenseName,
                url: licenseUrl,
                href: licenseHref,
                homepage: licenseHomepage,
                isExternal: licenseExternal,
                target: licenseTarget,
                cssClasses: {
                    'credits-content': true,
                    'is-hidden': parseInt(this.openedID, 10) !== licenseID
                }
            };
        }
    }
}
</script>

<style lang="scss" scoped>
@import '../scss/variables.scss';

.credits {
    &-list {
        margin-top: 2rem;
    }

    &-item {
        border-bottom: 1px solid var(--border-light-color);
        padding: 1.4rem 0;

        & > a {
            color: var(--link-secondary-color);
            float: right;
            font-size: 1.4rem;
            margin-left: 5rem;
            
            &:active,
            &:focus,
            &:hover {
                color: var(--link-secondary-hover-color);
            }
        }

        &:last-of-type {
            border-bottom: none;
        }
    }

    &-content {
        margin: 0;
        padding: 2rem;

        pre {
            white-space: pre-line;
        }
    }
}
</style>
