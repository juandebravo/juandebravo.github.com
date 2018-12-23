---
layout: post
title: "Plan B and unified Plan in WebRTC"
categories: [webrtc, html5, sdp]
---

A recurrent theme in new RTC technologies has been the need to
cleanly handle very large numbers of media flows.

The standard way of encoding this information in SDP is to have each
RTP flow (i.e., SSRC) appear on its own m-line.

One of the core problems with existing approaches that map media sources
to individual m= lines, is that m= lines, by their very nature, specify
information about how media should be transported. This means that if
N m= lines are created, N media ports (and their corresponding
STUN/TURN candidates) need to be allocated.

Intended to reduce the transport impact of using a large number of flows.

Unified plan: https://tools.ietf.org/html/draft-roach-mmusic-unified-plan-00
Plan B: https://tools.ietf.org/html/draft-uberti-rtcweb-plan-00

Semantic SDP: https://github.com/medooze/semantic-sdp-js

Jitsi: https://hacks.mozilla.org/2015/06/firefox-multistream-and-renegotiation-for-jitsi-videobridge/

SDP Interop: https://github.com/jitsi/sdp-interop

https://bugs.chromium.org/p/chromium/issues/detail?id=465349#c58

http://www.rtcbits.com/2014/09/using-native-webrtc-simulcast-support.html

https://webrtchacks.com/sfu-simulcast/

Unified plan:
    - Each media stream track is represented by its own unique m-line.
      This is a strict one-to-one mapping.
    - Each m-line is marked with an a=ssrc attribute to correlate it
      with its RTP packets.
    - Each m-line contains an MSID value to correlate it with a Media
      Stream ID and the Media Stream Track ID.
    - To minimize port allocation during a call, we rely on the BUNDLE
      [I-D.ietf-mmusic-sdp-bundle-negotiation] mechanism.

    DESIGN
    - Even with the use of BUNDLE, it is expensive to allocate ICE
        candidates for a large number of m-lines.
    - Bundle-Only M-Lines
    - Sender sets the port in the m-line to 0.  This indicates to old endpoints
      that the m-line is not to be negotiated.
    - Adds an a=bundle-only line.  This indicates to new endpoints that
      the m-line is to be negotiated if (and only if) bundling is used.
    - Correlating RTP Sources with m-lines. This correlation is performed
        by including a=ssrc attributes in the SDP.
    - Media Stream Tracks IDs are correlated with M-Lines directly by
   including an MSID in each m-line.

Plan B:
    - Each media source can be individually accepted or rejected by the recipient,
        and preferences on these track requests (e.g.
        the relative priority of the tracks) can also be provided.
    - Creates a hierarchy within SDP; a m= line defines an "envelope",
        specifying codec and transport parameters, and [RFC5576]
        a=ssrc lines are used to describe individual media sources within that envelope.
    - Supports large numbers (>> 100) of sources
    - Glareless add/removal of sources
    - No need to allocate ports for each source
    - Simple binding of MediaStreamTrack to SDP
    - Simple use of RTX and FEC streams within a single RTP session
    - Impossible to get "out-of-sync" (e.g.  getting RTX or FEC flow
      without its corresponding primary flow)
    - Does not require BUNDLE (although BUNDLE can still help)

    Note that at the wire level, RTP is handled identically to Plan A
   (which specifies separate m= line for each media source), after a
   successful BUNDLE, in that MSTs are all sent in a single RTP session,
   and demuxed by SSRC.  Therefore, Plan B could be converted to Plan A
   by a signaling-only gateway (although it would lose the advantages
   mentioned above).

   DESIGN
    - One m= line per [media-type, content] tuple
    - Regardless of how many sources are offered or used, only one m= line,
   by default, is used for each media type (e.g.  audio, video, or
   application).
   - Transmit sources identified by SSRC in signaling
   - Media sources are identified in signaling by SSRC, via [RFC5576]
   a=ssrc attributes.
   - Correlation of MediaStreamTracks with SSRCs is performed by the MSID attribute
   - When the offerer advertises its streams, the remote side can select
   which of them it wants to receive in its answer message.
   - Negotiation: `a=msid-semantic` indicates that you understand MSIDs.
