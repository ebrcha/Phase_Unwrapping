# Schofield–Zhu 2D Phase Unwrapping

This notebook implements a 2D phase-unwrapping method based on the Schofield–Zhu Fourier algorithm, based on the paper below. It is used for interferometric and holography data where the phase is wrapped within the interval $[-\pi, \pi)$, and the goal is to recover the continuous phase.

Ref: Schofield M A and Zhu Y 2003 Fast phase unwrapping algorithm for interferometric applications Opt. Lett.

## Purpose
A wrapped phase image contains discontinuities at values near $\pm \pi$, because phase is only known modulo $2\pi$. The goal of phase unwrapping is to reconstruct the continuous phase by correcting these jumps.

The method used here is based on the relationship between the wrapped phase and its Laplacian. Instead of using a local finite-difference method, the algorithm performs the calculation in the Fourier domain, which is often more stable and efficient for smooth phase fields.

## Key ideas
- The image is first extended by mirror symmetry to reduce boundary effects.
- The Laplacian is computed using Fourier differentiation.
- The phase is estimated by solving a Laplace-type equation in the frequency domain.
- Integer multiples of $2\pi$ are added or subtracted to recover the correct unwrapped phase.
- The final result is cropped back to the original image size.

## Author and date
- Author: Ebrahim Chalangar
- Institution: Denmark Technical University, DTU Nanolab
- Email: ebrahim.chalangar@gmail.com
- Date: 2026-09-10

## Overview of the code
The first function, `_mirror_symmetrize`, creates a larger image by reflecting the original data both horizontally and vertically. This produces a mirrored extension that reduces artificial boundary effects and improves the accuracy of Fourier-based differentiation.

The function `_spectral_laplacian` computes the 2D Laplacian using Fourier transforms instead of finite differences. In the Fourier domain, derivatives become multiplication by frequency values. The code creates frequency grids using `np.fft.fftfreq`, forms the squared wave-number term $k_x^2 + k_y^2$, multiplies the FFT of the image by $-(k_x^2 + k_y^2)$, and then applies the inverse FFT to recover the Laplacian.

The function `_inverse_spectral_laplacian` solves the equation $\nabla^2 \phi = \text{image}$ in the Fourier domain. Since the zero-frequency component corresponds to the undefined mean offset, the code sets the DC term to zero. This is physically reasonable because phase unwrapping cannot determine an arbitrary global phase offset.

The main function, `schofield_zhu_unwrap_2d`, validates the input, normalizes the wrapped phase into the principal range $[-\pi, \pi)$, extends the image by mirroring, and computes the Laplacian of the wrapped phase using the Fourier method. It then estimates the unwrapped phase and iteratively corrects it by integer multiples of $2\pi$ until the phase is consistent.

The update step is:

$$
\text{integer\_correction} = \mathrm{rint}\left(\frac{\text{phase\_estimate} - \text{current\_phase}}{2\pi}\right)
$$

and then the phase is updated by:

$$
\text{next\_phase} = \text{current\_phase} + 2\pi \times \text{integer\_correction}
$$

This works because wrapped phase values differ only by integer multiples of $2\pi$.

After convergence, the algorithm crops the mirrored region back to the original image size and optionally removes the mean value if `center_result=True`.

## Example comparison
The final code block compares the custom Schofield–Zhu unwrapping result with the output from `skimage`'s `unwrap_phase`. It creates a synthetic wrapped phase image from a grayscale input, unwraps it using both methods, and then displays a visual comparison of the original, wrapped, and reconstructed data. This helps confirm that the method is functioning correctly in practice.

This notebook is useful for understanding how Fourier-based phase unwrapping works and how it can be applied to experimental data in fields such as holography and interferometry.
