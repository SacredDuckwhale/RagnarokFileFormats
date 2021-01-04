# STR Format Specification

## Overview

STR files are a compiled (binary) form of animated, image-based effects used to display certain kinds of 2D effects in the game world. According to the author of RoBrowser, they are simply EZV compiled to binary (**TODO: Verify**).

In conjunction with hardcoded 3D effects that use geometrical primitives and Direct3D textures directly, as well as regular sprites (stored in SPR/ACT files), different types of effects can be realized.

They encode the following information:

* References to texture files (bitmaps)
* Data used to animate each layer
* Animation properties applicable to all textures

## Prerequisites

In order to understand the STR file format, some familiarity with the following concepts is required:

* It's recommended to read the [SPR](SPR.MD) and [ACT](ACT.MD) specifications first, as sprite animations share many key concepts with STR effects
* [Keyframe animation](https://en.wikipedia.org/wiki/Key_frame) is essentially what's being used by both sprites and effects, in a slightly different fashion

## Effects vs. Sprites

While storing effects in a separate file format seems handy, they differ from SPR and ACT files (that can also be abused to create effects) in that they don't contain the actual image data. Instead, they reference files locally, i.e. inside the ``data/texture/effect`` folder. This is where both the STR files and the bitmaps they use are stored.

From a technical POV, it appears that sprites are simply meshes rendered in billboard mode (always facing the camera) and then are modified to account for camera perspective, while STR effects are rendered either as projected textures (for AOE/ground effects) or similarly as "actors" in the game world but without the complex logic required to handle look/face directions (**TODO: Is this actually true?**). More research is needed to confirm this, however.

I assume these effects are unable to play sounds, as the ACT data contains event emitters that are forwarded to the engine (example: mvp.act)? 2d/3d primitive RagEffects CAN play sounds, but these are hardcoded? STR effects may also work like this, but use the STR instead of SPR/ACT for billboard effects/no sound/no layering,anchor points?



**TODO: Confirm by checking mvp.act  and RagEffect**

## Layout

The file structure differs slightly with each file version, as newer versions added more features. Please consult the tables below for a detailed specification.

**TODO: Note the version byte order**

**TODO: Layout for the different versions**

## Tools

There are only a few tools available to parse STR files, as far as I know:

* [RO STR Viewer](https://herc.ws/board/topic/17268-rostrviewer-reupload): Looks a bit dated, but contains all the information as far as I can tell
* https://github.com/skardach/ro-str-viewer: I actually didn't care enough to make it run, but an open-source reference implementation is always nice to have
* RoBrowser (see [Tools.md](Tools.md)) has an Effect viewer that appears to be fully functional

## WIP (Delete later)

* cLayer can be 128 at most, but devil.str has cLayer = 145??
* cEndLayer in KANICLIP is unused? (overwritten in ``CRagEffect::LoadEzEffect` and ``CRagEffect::InitEZ2STRFrame```)
* KSTR version is fixed to 148? (0x0094)
* What about the KAC_XFORMDATA (Keyframe Animation Clip?)
* nFPS is zero and unused?
* cFrame is 60 and unused?
* Review RenderAniClip
* Review m_isLayerDrawn
* Research EZV format and see if it matches what we expect (compiled/binary EZV = STR)
* Review and explain concepts: Keyframe/Animation Clip, Layer, frame, transform?
* Research exact differences between ACT/SPR Sprites and STR EzEffects
* Can they play sound effects? MVP effect uses SPR, so probably no (event emitter of the ACT data is used for this, I think)

In ``CEZeffect :: OnInit()`` we find this code:

````
m_cEndLayer = m_aniClips->cLayer;
if (m_aniClips->aLayer[0].cAniKey == 0) m_cEndLayer--;
````
mtpreset property refers to alpha blend mode? (seems to be ignored in RoBrowser)

	SetMultiTextureMode:
		switch (nMode) {
			// Modulate Alpha
			case 0:
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );

				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE);

				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_MODULATE );

				m_device->SetTextureStageState(1, D3DTSS_COLOROP, D3DTOP_DISABLE);
				m_device->SetTextureStageState(1, D3DTSS_ALPHAOP, D3DTOP_DISABLE);
				break;

		case 1:
			// Modulate
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );

				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_DISABLE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_DISABLE );
			}
			break;

			// Modulate Alpha
		case 2:
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );

				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_DISABLE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_DISABLE );
			}
			break;

		case 3:
			// Add
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_ADD );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_DISABLE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_DISABLE );
			}
			break;

		case 4:
			// Decal Alpha
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_BLENDTEXTUREALPHA );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_DISABLE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_DISABLE );
			}
			break;

		case 5:
			// Colored light map
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 6:
			// Inverse colored light map
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE | D3DTA_COMPLEMENT );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE | D3DTA_COMPLEMENT );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 7:
			// Single channel light map
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_SELECTARG1 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE | D3DTA_ALPHAREPLICATE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 8:
			// Modulate and late add
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_ADD );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 9:
			// Linear blend using texture alpha
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_BLENDTEXTUREALPHA );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 10:
			// Linear blend using diffuse alpha
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_BLENDDIFFUSEALPHA );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 11:
			// Multitexture add w/smooth saturation
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_ADD );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 12:
			// Multitexture subtract
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE | D3DTA_COMPLEMENT );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_ADD );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 13:
			// Add Diffuse to light map then modulate
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_ADD );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 14:
			// Detail modulate
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_MODULATE2X );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;

		case 15:
			// Detail add
			{
				m_device->SetTextureStageState( 0, D3DTSS_TEXCOORDINDEX, 0 );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 0, D3DTSS_COLOROP,   D3DTOP_MODULATE );
				m_device->SetTextureStageState( 0, D3DTSS_COLORARG2, D3DTA_DIFFUSE );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG2 );
				m_device->SetTextureStageState( 0, D3DTSS_ALPHAARG2, D3DTA_DIFFUSE );

				m_device->SetTextureStageState( 1, D3DTSS_TEXCOORDINDEX, 1 );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_COLOROP,   D3DTOP_ADDSIGNED );
				m_device->SetTextureStageState( 1, D3DTSS_COLORARG2, D3DTA_CURRENT );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAARG1, D3DTA_TEXTURE );
				m_device->SetTextureStageState( 1, D3DTSS_ALPHAOP,   D3DTOP_SELECTARG1 );
			}
			break;
		}
