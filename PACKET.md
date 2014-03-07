# IEEE Packet format
Some notes on the 802.11 format

## RadioTap Header
[Radiotap](http://www.radiotap.org/) is a de facto standard for 802.11 frame injection and reception.
```C
        struct ieee80211_radiotap_header {
        u_int8_t        it_version;     /* set to 0 */
        u_int8_t        it_pad;
        u_int16_t       it_len;         /* entire length */
        u_int32_t       it_present;     /* fields present */
} __attribute__((__packed__));
```
[it_present](http://www.radiotap.org/defined-fields) is a bitmask. 
It gives us various info like:
* Date rate
* Channel frequency
* Channel type
* SSI Signal
* SSI Noice
* Antenna

Parsing this field is interesting. There is a python implementation [here](https://code.google.com/p/python-radiotap). The effective payload is the part after it_len bytes of the packet. This is the 80211 frame.

## 802.11 Frame
An 802.11 frame could be one of three:
* Management Frame
* Control Frame
* Data Frame

### Management Frame

#### Capability
```
/* Capabilities Field */
typedef struct tagCAPABILITY_FIELD
{
    u_int16_t       ESS:                1;
    u_int16_t       IBSS:               1;
    u_int16_t       CFPollable:         1;
    u_int16_t       CFPollRequest:      1;
    u_int16_t       Privacy:            1;
    u_int16_t       ShortPreamble:      1;
    u_int16_t       PBCC:               1;
    u_int16_t       ChannelAgility:     1;
    u_int16_t       SpectrumMgmt:       1;
    u_int16_t       QOS:                1;
    u_int16_t       ShortSlotTime:      1;
    u_int16_t       APSD:               1;
    u_int16_t       Rsvd3:              1;
    u_int16_t       DSSS_OFDM:          1;
    u_int16_t       BlockAck:           1;
    u_int16_t       ImmediateBlockAck:  1;
} CAPABILITY_FIELD, *PCAPABILITY_FIELD;
```
[802.11 management frame capability field bits](http://dox.ipxe.org/group__ieee80211__capab.html)

#### Management information element payloads
```
/*
 * Management information element payloads.
 */

enum {
    IEEE80211_ELEMID_SSID       = 0,
    IEEE80211_ELEMID_RATES      = 1,
    IEEE80211_ELEMID_FHPARMS    = 2,
    IEEE80211_ELEMID_DSPARMS    = 3,
    IEEE80211_ELEMID_CFPARMS    = 4,
    IEEE80211_ELEMID_TIM        = 5,
    IEEE80211_ELEMID_IBSSPARMS  = 6,
    IEEE80211_ELEMID_COUNTRY    = 7,
    IEEE80211_ELEMID_CHALLENGE  = 16,
    /* 17-31 reserved for challenge text extension */
    IEEE80211_ELEMID_PWRCNSTR   = 32,
    IEEE80211_ELEMID_PWRCAP     = 33,
    IEEE80211_ELEMID_TPCREQ     = 34,
    IEEE80211_ELEMID_TPCREP     = 35,
    IEEE80211_ELEMID_SUPPCHAN   = 36,
    IEEE80211_ELEMID_CSA        = 37,
    IEEE80211_ELEMID_MEASREQ    = 38,
    IEEE80211_ELEMID_MEASREP    = 39,
    IEEE80211_ELEMID_QUIET      = 40,
    IEEE80211_ELEMID_IBSSDFS    = 41,
    IEEE80211_ELEMID_ERP        = 42,
    IEEE80211_ELEMID_HTCAP      = 45,
    IEEE80211_ELEMID_RSN        = 48,
    IEEE80211_ELEMID_XRATES     = 50,
    IEEE80211_ELEMID_HTINFO     = 61,
    IEEE80211_ELEMID_TPC        = 150,
    IEEE80211_ELEMID_CCKM       = 156,
    IEEE80211_ELEMID_VENDOR     = 221,  /* vendor private */

    /*
     * 802.11s IEs
     * NB: On vanilla Linux still IEEE80211_ELEMID_MESHPEER = 55,
     * but they defined a new with id 117 called PEER_MGMT.
     * NB: complies with open80211
     */
    IEEE80211_ELEMID_MESHCONF   = 113,
    IEEE80211_ELEMID_MESHID     = 114,
    IEEE80211_ELEMID_MESHLINK   = 115,
    IEEE80211_ELEMID_MESHCNGST  = 116,
    IEEE80211_ELEMID_MESHPEER   = 117,
    IEEE80211_ELEMID_MESHCSA    = 118,
    IEEE80211_ELEMID_MESHTIM    = 39, /* XXX: remove */
    IEEE80211_ELEMID_MESHAWAKEW = 119,
    IEEE80211_ELEMID_MESHBEACONT    = 120,
    /* 121-124 MMCAOP not implemented yet */
    IEEE80211_ELEMID_MESHPANN   = 125, /* XXX: is GANN now, not used */
    IEEE80211_ELEMID_MESHRANN   = 126,
    /* 127 Extended Capabilities */
    /* 128-129 reserved */
    IEEE80211_ELEMID_MESHPREQ   = 130,
    IEEE80211_ELEMID_MESHPREP   = 131,
    IEEE80211_ELEMID_MESHPERR   = 132,
    /* 133-136 reserved */
    IEEE80211_ELEMID_MESHPXU    = 137,
    IEEE80211_ELEMID_MESHPXUC   = 138,
    IEEE80211_ELEMID_MESHAH     = 60, /* XXX: remove */
};

```

This is parsed into ie_<number> in dpkt ( see unpack_ies in ieee80211.py )
Each IE has a length, and data. Depending on the type, the data will haveto be parsed accordingly.
For example, consider IEEE80211_ELEMID_RSN. It deals with cipher information for the channel.

### Control Frame
### Data Frame

## References
* [radiotap-headers.txt](https://www.kernel.org/doc/Documentation/networking/radiotap-headers.txt)
* [802-11-Frame_E_C.pdf](http://www.studioreti.it/slide/802-11-Frame_E_C.pdf)
* [Wikipedia - IEEE 802.11](http://en.wikipedia.org/wiki/IEEE_802.11)
