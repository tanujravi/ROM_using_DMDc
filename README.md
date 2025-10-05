# List of Tests done

## 01_model.ipynb

Purpose & Observations:

Fits a DMDc model on the case where Amplitude = 0.5 & Frequency = 3.0 by using train_test_split ratio of 0.75. In this case a "svd_rank" of 20 is used based on plotting the Singular values (from SVD of data matrix). Reconstruction of the data matrix is done using a library function from pydmd. The reconstructed data is visualized and compared with original data matrix at different time steps. This shows a good match visually. The reconstruction error is obtained quantitatively by taking the L2 norm of the error and then normlizing it. The reconstruction error in every timestep is then plotted. The model is then tested on the test data from the train-test split. This can be done by evolving over time using the DMDc operator. One way to do this is to use "predict" function from pydmd library. But when this was done in this case it led to excess memory usage and maxing out of RAM. So, the time evolution was implemented using reduced order "A_tilde" & "B_tilde" matrices and then projecting it to full space to get the results. The results are again visualized by comparing contour plots of original and predicted flow at different times and is quantified by taking the L2 norm of the error. Finally, to check the effectiveness of DMDc operator the model is tested for unforced case by setting "u" vector in the DMDc operator to 0. The results are visualized in the same way as mentioned above.

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   0.5;
        frequency   3.0;
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * ($amplitude)*sin(2*pi()*($frequency)*time())";
    }
```

Conclusion & Next steps:

The results show very good accuracy for reconstruction and prediction at the same frequency, but when tested for the unforced case the error increases. Moreover, the dynamics of the system needs to be studies that is the A & B operators and their effects need to be studied individually as their effects are not evindent in these results.

## 02_dynamics_compare.ipynb

Purposes & Observations:

The aim of this study is to check the differences between the dynamics behavior shown by the DMD and DMDc operator, that is the eigenvalue and mode shapes of the A operator. The ideal svd_rank is now determined by doing a parametric study calculating the reconstruction and prediction error at different ranks. For further analysis the rank chosen is the one at which the reconstruction error becomes nearly the same. After selecting the optimum rank, DMD and DMDc models are fit for the same data that is the pressure state for Amplitude = 0.5 & Frequency = 3.0. The most dominant modes are selected so that the frequencies have the highest integral contribution.

Conclusion & Next steps:

The Eigenvalues and mode shapes are of both the DMD and DMDc seems to be same. This is interesting and may be attributed to the fact that the forcing frequency is sinusoidal in nature and the DMD operator is able capture this. There are several changes to be made in the notebook like make making the plots side by side for comparison and choosing right timestep for frequency (as it is currently scaled with timestep of value 1.0), these changes are made in the further notebooks.

## 03_dmdc_model_predict.ipynb

Purposes & Observations:

The aim of this study is to check how well the dmdc model performs when fit with data from forcing frequency (Frequency = 1.5) which is in synchronisation region and then use the DMDc operator to check if the model performs well for forcing frequency (Frequency = 2.1)in non-synchronisation region. This is first visually compared by comparing the contours of original flow and predicted flow and then computing the L2 norm for time-varying error. The "B" operator is also visualized by plotted it index wise and also by plotting a contour of it.

Conclusion and Next steps:

The prediction accuracy for the non-synchronisation is relatively low, which is seen from the contour plots and the L2 error which oscillates in time between 0.2 to 0.7. The absolutes values in B matrix is very low as seen from the "B matrix" plot, this suggests that the influence of the B operator is very low to the overall prediction results.

## 04_dynamics_compare_synchronisation.ipynb

Purposes & Observations:

Aim of this study is to to check the eigenvalues and mode shapes of A operator for case in which the forcing frequency lies in the synchronisation region. This done for both DMD and DMDc models and then compared to check for the performance. Corrections to previous notebooks are made to subtract the mean from original pressure data (this makes it so that we don't get the mean flow mode), correctly scaled dominant frequencies, dominant integral contributions, Prediction and Reconstruction error comparison, Eigenvalues and Eigenmodes comparison.

Conclusion & Next Steps:

The DMD model performs much better than the DMDc model, which is seen from the prediction and reconstruction error v/s rank  plots. This result is surprising as DMDc model has more degree of freedoms to capture the model better. From the mode shapes and domninant frequencies of DMD model it is observed that it is able to capture the forcing frequency of 3 Hz and also it's harmonics. Same is the case with DMDc model. The eignevalues for both DMD and DMDc models are mostly the same except one wheich is decaying in case of DMDc model and oscillatory in case of DMD model.

## 05_dynamics_compare_no_synchronisation.ipynb

Purposes & Observations:

Aim of this study is to to check the eigenvalues and mode shapes of A operator for case in which the forcing frequency lies in the non-synchronisation region. This done for both DMD and DMDc models and then compared to check for the performance.

Conclusion & Next Steps:

The DMDc model performs same than the DMD model, which is seen from the prediction and reconstruction error v/s rank  plots. This result is surprising as DMDc model has more degree of freedoms to capture the model better. From the mode shapes and domninant frequencies of DMD model it is observed that it is able to capture the forcing frequency of 1.5 Hz and also it's harmonics as well as the natural frequency of flow. Same is the case with DMDc model. The eignevalues for both DMD and DMDc models are mostly the same except one wheich is decaying in case of DMDc model and oscillatory in case of DMD model.

## 06_dynamics_compare_linear_forcing_signal.ipynb

Purposes & Observations:

Aim of this study is to to check the eigenvalues and mode shapes of A operator for case in which the forcing signal is not sinusoidal but linearly increases and decreases along with the sin curve like an oscillating ramp function. This signal is chosen as it seen that the DMD model is able to capture the sinusoidal signal and a signal is required such that it is difficult for DMD to capture. Then the difference between DMD & DMDc can be observed. The signal is tested for both DMD and DMDc models and then compared to check for the performance.

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   0.5;
        frequency   1.5;
        variables
        (
            "time_period  = 1/($frequency)"
            "shifted_time = time() - 1/(4*($frequency))"
        );
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * (($amplitude)-4*($amplitude)*(0.5-mag(shifted_time/time_period - floor(shifted_time/time_period)-0.5)))";
    }
```

Conclusion & Next Steps:

The DMDc model performs similar than the DMD model, which is seen from the prediction and reconstruction error v/s rank  plots. From the mode shapes and domninant frequencies of DMD model it is observed that it is still able to capture the forcing frequency of 1.5 Hz and also it's harmonics as well as the natural frequency of flow. Same is the case with DMDc model. The eignevalues for both DMD and DMDc models are mostly the same except one wheich is decaying in case of DMDc model and oscillatory in case of DMD model.

## 07_dynamics_compare_linear_increasing_forcing.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that has a linearly increasing forcing frequency (chirp signal). This signal is chosen as it seen that the DMD model is able to capture forcing frequncy even  when an oscillating ramp function is used.

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   0.5;
        frequency   1.5;
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * ($amplitude)*sin(2*pi()*($frequency)*(1/3)*time()*time())";
    }
```

Conclusion & Next Steps:

Here, it is seen that both DMD & DMDc models perform poorly but the DMDc a sligthly better, which is seen from the prediction and reconstruction error v/s rank  plots.
Both the DMD and DMDc models capture different frequency modes and shapes. The DMDc models has quite a few exploding eigenvalues (value>1). Seems like the B matrix has much more of an influence with larger values compared to previous tests.

## 08_dynamics_compare_random_signal.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that changes it's forcing frequency randomly. This signal is chosen as a benchmark to compare the DMD and DMDc models.

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   2.0;
        frequency   6.0;
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * (time() >= 4.0 && time() < 4.8 ? -0.53487*(time() - 4): time() >= 4.8 && time() < 5.6 ? 1.68*time() - 8.4913 :time() >= 5.6 && time() < 6.4 ? -0.4687*time() + 3.541 : time() >= 6.4 && time() < 7.2 ? 0.5412*time() -2.923 : time() >= 7.2 && time() <= 8 ? -1.2175*time() + 9.74 : 0)";
    }
```

Conclusion & Next Steps:

Here, it is seen that both DMD & DMDc models perform poorly and the DMDc diverges for a few ranks between 28 to 35, which is seen from the prediction and reconstruction error v/s rank  plots.
Both the DMD and DMDc models capture similar frequency modes and shapes. The values in the B matrix are small and influence is much smaller compared to previous tests.

## 09_dynamics_compare_switch_signal.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that switches on and off and it is sinusoidal when it is on.

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   0.5;
        frequency   1.5;
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * (time() > 5 ? (time() < 7 ? ($amplitude)*sin(2*pi()*($frequency)*time()):0) : 0)";
    }
```

Conclusion & Next Steps:

Here, it is seen that both DMD & DMDc models perform similarly and poorly with high errors, which is seen from the prediction and reconstruction error v/s rank  plots.
Both the DMD and DMDc models capture similar frequency modes and shapes. The values in the B matrix are small and influence is much smaller compared to previous tests. The DMD operator has an exploding eigenvalue and the DMDc operator has only decaying and oscillating eigenvalues.

## 10_dynamics_compare_freq_ampl_modulation.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that oscillates sinusoidally in amplitude and forcing frequency (F/A modulation).

OpenFOAM Implementation:

```
cylinder
    {
        type        exprFixedValue;
        value       $internalField;
        amplitude   0.5;
        frequency   0.125;
        valueExpr   "(vector(0, 0, 1) ^ face()) / area() * ($amplitude)*sin(2*pi()*($frequency)*time())*sin(2*pi()*($frequency)*46*sin(2*pi()*($frequency)*time()))";
    }
```

Conclusion & Next Steps:

Here, it is seens that both DMD & DMDc models perform similarly and poorly with high errors, which is seen from the prediction and reconstruction error v/s rank  plots.
Both the DMD and DMDc models capture similar frequency modes and shapes. The values in the B matrix are small and influence is much smaller compared to previous tests.

## 11_dynamics_compare_inlet_velocity_switch_signal.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that changes the inlet velocity sinusoidally. The Reynolds number varies from 90 to 110.

OpenFOAM Implementation:

```
inlet
{
    type            uniformFixedValue;
    uniformValue
    {
            type            expression;
            variables
            (
                    "val = time() > 5 ? (time() < 7 ? 1 + 0.1*sin(2*pi()*1.5*time()) : 1) : 1"
            );
            expression
            #{
                    vector(val, 0, 0)
            #};
    }
}
```

Conclusion & Next Steps:

Here, it is seen that both DMD & DMDc models perform similarly and poorly with high errors, which is seen from the prediction and reconstruction error v/s rank  plots.
DMDc model has quite a few exploding eigenvalues.

## 12_dynamics_compare_inlet_freq_ampl_mod.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that changes the inlet velcity such that it oscillates sinusoidally in amplitude and forcing frequency (F/A modulation).

OpenFOAM Implementation:

```
inlet
{
    type            uniformFixedValue;
    uniformValue
    {
            type            expression;
            variables
            (
            "val = (time() >= 4 ? 1 + 0.1 * sin(2*pi()*0.125*time()) * sin(2*pi()*0.125 * 46 * sin(2*pi()*0.125*time())) : 1)"
            );

            expression
            #{
                    vector (val, 0, 0)
            #};
    }
}
```

Conclusion & Next Steps:

Here, it is seen that both DMD & DMDc models perform similarly and poorly with high errors, which is seen from the prediction and reconstruction error v/s rank  plots.
DMD model has quite a few exploding eigenvaues. The values in the B matrix are small and influence is much smaller compared to previous tests.

## 13_dynamics_compare_inlet_ramp.ipynb

Purposes & Observations:

Aim of this study is to to check the performance of DMD and DMDc model for a forcing signal that changes the inlet velcity such that it ramps from a velocity of 1 to 1.1.

OpenFOAM Implementation:

```
inlet
{
    type            uniformFixedValue;
    uniformValue
    {
            type            expression;
            variables
            (
            "val = time() >= 4 ? (0.1/4)*time() + 0.9 : 1"
            );
            expression
            #{
                    vector (val, 0, 0)
            #};
    }
}
```

Conclusion & Next Steps:

Here, it is seen that both DMDc models perform and poorly compared to DMD model with high errors, which is seen from the prediction and reconstruction error v/s rank  plots.
DMD and DMDc models have quite a few exploding eigenvaues. The values in the B matrix are small and influence is much smaller compared to previous tests.

## 14_dynamics_compare_extended_state_vector.ipynb

Purposes & Observations:

Aim of this study is to use an extended state vector and test the performance. In the previous tests, only pressure was used in the state vector and the effect of velocity also needs to be tested. This is done in two steps, first only the velcity in x and y direction is used and then a combined state vector of pressure and velocity is used to fit the model. A normal sinusoidal actuation with Forcing frequency = 3.0 and Amplitude = 0.5 is used.

Conclusion & Next Steps:

It is observed that the DMDc model performs slightly better when only Velocity is considered in the state vector. When the extended state vector also includes the pressure, the DMD model performs better in terms of reconstruction error.

## 15_dynamics_compare_including_noise.ipynb

Purposes & Observations:

Aim of this study is to test the performance of DMD and DMDc model when white noise is added to the extended state vector.

Conclusion & Next Steps:

The DMD and DMDc model perform very similarly, with DMD model performing slightly better in terms of error.