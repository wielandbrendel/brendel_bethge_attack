=======================
Brendel & Bethge attack
=======================

This repo contains a few short examples on how to use the `adversarial attacks by Brendel & Bethge <https://arxiv.org/abs/1907.01003>`_ which have been published at NeurIPS 2019 and are state-of-the-art on several Lp metrics (L0, L1, L2, L-infinity).

The reference implementation is available in `Foolbox <https://github.com/bethgelab/foolbox>`_. We build a thin wrapper, called `CleverFool <https://github.com/wielandbrendel/cleverfool>`_, to make the attack also available in CleverHans. The wrapper can be installed with

.. code-block:: python

   pip install cleverfool

TODO: CleverFool will be updated soon to support the new Foolbox 3.

Usage notes
-----------

Please follow the runnable and concise examples in the notebooks to see Brendel & Bethge attacks in action. There is only one subtlety one might want to take care of: the Brendel & Bethge attacks need a starting point that is already adversarial (but might be very far away from the clean image). By default the Brendel & Bethge employs a gradient-free noise attack if no starting points are given. However, sometimes this noise attack might fail to find an adversarial, thus making the rest of the attack fail as well.

A solution to this problem is to give suitable starting points by choosing other clean samples from the data set. To make this easy Foolbox implements an attack call DatasetAttack. This pseudo-attack is initialised by passing several batches of clean data from which it then chooses suitable starting images without any further user intervention. Below you find a minimal example (also available in full in the Jupyter notebooks).

.. code-block:: python

   # get some clean data
  images, labels = fb.utils.samples(fmodel, dataset='imagenet', batchsize=20)

  # split images and labels into 'batches' 
  batches = [(images[:10], labels[:10]), (images[10:], labels[10:])]

  # create attack that picks adversarials from given dataset of samples
  init_attack = fb.attacks.DatasetAttack()

  # feed clean data into the pseudo-attack (in most applications, especially in targeted 
  # scenarios, you will need to feed many batches)
  init_attack.feed(fmodel, batches[0][0])   # feed 1st batch of inputs
  init_attack.feed(fmodel, batches[1][0])   # feed 2nd batch of inputs

  # apply the Brendel & Bethge attack
  attack = fb.attacks.L2BrendelBethgeAttack(init_attack=init_attack, steps=100)
  adversarials = attack(
      fmodel,
      images,
      criterion=fb.criteria.Misclassification(labels),
      epsilons=1
  )
