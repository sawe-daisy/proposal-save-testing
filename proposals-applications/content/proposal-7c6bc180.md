'use client';

import MemberOnlyAccessCard from '@/app/my/\_component/MemberOnlyAccessCard';

import Button from '@/components/atoms/Button';

import Modal from '@/components/atoms/Modal';

import Title from '@/components/atoms/Title';

import TopNav from '@/components/Navigation/TabNav';

import CardanoLoaderSVG from '@/components/ui/CardanoLoaderSVG';

import { routes } from '@/config/routes';

import { useApp } from '@/context';

import { useMemberValidation } from '@/hooks';

import {

  adaToLovelace,

  dbUtxoToMeshUtxo,

  findTokenUtxoByMemberUtxo,

  getCatConstants,

  getProvider,

  parseAdaInput,

  smoothScrollToElement,

} from '@/utils';

import { resolveTxHash } from '@meshsdk/core';

import {

  ProposalData,

  proposalMetadata,

  UserActionTx,

} from '@sidan-lab/cardano-ambassador-tool';

import { useEffect, useRef, useState } from 'react';

import DetailsTab from './components/DetailsTab';

import FundsTab from './components/FundsTab';

import ReviewTab from './components/ReviewTab';

type ProposalFormData = ProposalData & {

  description: string;

};

export default function SubmitProposalPage() {

  const { userWallet, memberUtxo, userAddress, showTxConfirmation } = useApp();

  const { isMember, isLoading: memberLoading } = useMemberValidation();

  const \[activeTab, setActiveTab\] = useState('details');

  const \[isSubmitting, setIsSubmitting\] = useState(false);

  const \[markdownData, setMarkdownData\] = useState&lt;any&gt;({});

  const \[showConfirmation, setShowConfirmation\] = useState(false);

  const \[showError, setShowError\] = useState(false);

  const \[error, setError\] = useState&lt;string&gt;('');

  const \[githubFilename, setGithubFilename\] = useState&lt;string&gt;('');

  const descriptionEditorRef = useRef&lt;any&gt;(null);

  const impactEditorRef = useRef&lt;any&gt;(null);

  const objectivesEditorRef = useRef&lt;any&gt;(null);

  const milestonesEditorRef = useRef&lt;any&gt;(null);

  const budgetBreakdownEditorRef = useRef&lt;any&gt;(null);

  const impactOnEcosystemEditorRef = useRef&lt;any&gt;(null);

  const scrollTargetRef = useRef&lt;HTMLDivElement&gt;(null);

  const ORACLE_TX_HASH = [process.env.NEXT](http://process.env.NEXT)\_PUBLIC_ORACLE_TX_HASH!;

  const ORACLE_OUTPUT_INDEX = parseInt(

    [process.env.NEXT](http://process.env.NEXT)\_PUBLIC_ORACLE_OUTPOUT_INDEX || '0',

  );

  const blockfrost = getProvider();

  const \[formData, setFormData\] = useState&lt;ProposalFormData&gt;({

    title: '',

    description: '',

    url: '',

    fundsRequested: '',

    receiverWalletAddress: '',

    submittedByAddress: userAddress || '',

    status: 'pending',

  });

  const tabs = \[

    { id: 'details', label: 'Details' },

    { id: 'funds', label: 'Funds' },

    { id: 'review', label: 'Review' },

  \];

  useEffect(() =&gt; {

    if (

      userAddress &&

      (!formData.submittedByAddress || formData.submittedByAddress === '')

    ) {

      setFormData((prev) =&gt; ({

        ...prev,

        submittedByAddress: userAddress,

      }));

    }

  }, \[userAddress\]);

  if (!isMember) {

    return (

      &lt;MemberOnlyAccessCard

        title="Members Only: Submit Proposals"

        description="Only approved Cardano Ambassadors can submit proposals to the community. Join our ambassador program to contribute your ideas and help shape the Cardano ecosystem."

        feature="submit proposals"

      /&gt;

    );

  }

  const handleInputChange = (field: keyof ProposalFormData, value: string) =&gt; {

    setFormData((prev) =&gt; ({

      ...prev,

      \[field\]: value,

    }));

  };

  const handleSubmit = async () =&gt; {

    let filename = '';

    try {

      setIsSubmitting(true);

      console.log("=== STARTING PROPOSAL SUBMISSION ===");

      const saveResponse = await fetch('/api/proposal-content', {

        method: 'POST',

        headers: { 'Content-Type': 'application/json' },

        body: JSON.stringify({

          title: formData.title,

          description: formData.description,

          submitterAddress: userAddress,

        }),

      });

      if (!saveResponse.ok) {

        const errorData = await saveResponse.json();

        throw new Error(

          errorData.details || 'Failed to save proposal to GitHub',

        );

      }

      const saveData = await saveResponse.json();

      filename = [saveData.data](http://saveData.data).filename;

      setGithubFilename(filename);

      if (!memberUtxo) {

      console.error("ERROR: memberUtxo is null/undefined from context");

      throw new Error('No membership application UTxO found for this address');

    }

    

    console.log("2. memberUtxo from context:", memberUtxo);

    console.log("3. memberUtxo address:", memberUtxo.address);

    

    // Get constants

    const catConstants = getCatConstants();

    const memberTokenPolicyId = [catConstants.scripts.member.mint](http://catConstants.scripts.member.mint).hash;

    console.log("4. Member token policy ID:", memberTokenPolicyId);

    

    const mbrUtxo = dbUtxoToMeshUtxo(memberUtxo);

    console.log("5. mbrUtxo after dbUtxoToMeshUtxo:", mbrUtxo);

    console.log("6. mbrUtxo output address:", mbrUtxo?.output?.address);

    console.log("7. mbrUtxo assets:", mbrUtxo?.output?.amount);

    

    if (!userWallet) {

      throw new Error('Wallet not connected');

    }

    // ============== FIXED SECTION ==============

    console.log("=== FINDING IDENTITY TOKEN ===");

    

    // Use the existing function - it knows what token to look for

    const tokenUtxo = await findTokenUtxoByMemberUtxo(mbrUtxo);

    if (!tokenUtxo) {

      console.error("findTokenUtxoByMemberUtxo returned null");

      

      // Additional debug info

      console.log("Debug mbrUtxo:", {

        address: mbrUtxo.output.address,

        hasPlutusData: !!mbrUtxo.output.plutusData,

        datumPreview: mbrUtxo.output.plutusData?.substring(0, 100) + "..."

      });

      

      throw new Error(

        'Could not find the required identity token in your wallet. ' +

        'Please ensure you are using the same wallet that you used when ' +

        'applying for membership, and that you have the required token.'

      );

    }

    

    console.log("8. tokenUtxo found:", {

      txHash: tokenUtxo.input.txHash,

      address: tokenUtxo.output.address,

      assets: tokenUtxo.output.amount

    });

    // ============== END FIXED SECTION ==============

    const oracleUtxos = await blockfrost.fetchUTxOs(

      ORACLE_TX_HASH,

      ORACLE_OUTPUT_INDEX,

    );

    console.log("8. oracleUtxos fetched:", oracleUtxos.length);

    

    const oracleUtxo = oracleUtxos\[0\];

    console.log("9. oracleUtxo:", oracleUtxo);

    if (!oracleUtxo) {

      console.error("ERROR: No oracle UTxO found");

      throw new Error('Failed to fetch required oracle UTxO');

    }

    // Already have catConstants from above

    console.log("10. Cat Constants member script address:", catConstants.scripts.member.spend.address);

    

    // Check if mbrUtxo is at correct address

    const memberScriptAddress = catConstants.scripts.member.spend.address;

    const isMbrAtScriptAddress = mbrUtxo?.output?.address === memberScriptAddress;

    console.log("11. Is mbrUtxo at script address?", isMbrAtScriptAddress);

    console.log("   Expected:", memberScriptAddress);

    console.log("   Actual:", mbrUtxo?.output?.address);

    

    // Check if mbrUtxo contains member token (already have memberTokenPolicyId)

    const hasMemberTokenInMbr = mbrUtxo?.output?.amount?.some(asset =&gt; 

      asset.unit.includes(memberTokenPolicyId)

    );

    console.log("12. Does mbrUtxo contain member token?", hasMemberTokenInMbr);

    

    // Check if tokenUtxo contains member token

    const hasMemberTokenInToken = tokenUtxo?.output?.amount?.some(asset =&gt;

      asset.unit.includes(memberTokenPolicyId)

    );

    console.log("13. Does tokenUtxo contain member token?", hasMemberTokenInToken);

    

    console.log("14. Creating UserActionTx...");

    const userAction = new UserActionTx(

      userAddress!,

      userWallet!,

      blockfrost,

      catConstants,

    );

      const cleanAdaAmount = parseAdaInput(formData.fundsRequested);

      const lovelaceAmount = adaToLovelace(cleanAdaAmount);

      const githubUrl = `https://github.com/${process.env.NEXT_PUBLIC_GITHUB_REPO}/blob/${process.env.NEXT_PUBLIC_GITHUB_BRANCH}/proposals-applications/content/${filename}`;

      

      console.log({ filename, githubUrl });

      

      const metadataFormData: ProposalData = {

        title: formData.title,

        url: githubUrl,

        fundsRequested: lovelaceAmount.toString(),

        receiverWalletAddress: formData.receiverWalletAddress,

        submittedByAddress: formData.submittedByAddress,

        status: formData.status,

      };

      const metadata = proposalMetadata(metadataFormData);

      const result = await userAction.proposeProject(

        oracleUtxo,

        tokenUtxo,

        mbrUtxo,

        lovelaceAmount,

        formData.receiverWalletAddress,

        metadata,

      );

      const txHash = resolveTxHash(result.txHex);

      showTxConfirmation({

        txHash,

        title: 'Proposal Submitted',

        description:

          'Your proposal has been submitted. Please wait for blockchain confirmation.',

        onConfirmed: () =&gt; {

          setShowConfirmation(true);

        },

        onTimeout: async () =&gt; {

          if (filename) {

            await fetch('/api/proposal-content', {

              method: 'DELETE',

              headers: { 'Content-Type': 'application/json' },

              body: JSON.stringify({ filename }),

            }).catch((err) =&gt;

              console.error('Failed to rollback GitHub file:', err),

            );

          }

          setError(

            'Transaction confirmation timed out. Your proposal may still be processed.',

          );

          setShowError(true);

        },

      });

    } catch (error: any) {

      if (filename) {

        await fetch('/api/proposal-content', {

          method: 'DELETE',

          headers: { 'Content-Type': 'application/json' },

          body: JSON.stringify({ filename }),

        }).catch((err) =&gt;

          console.error('Failed to rollback GitHub file:', err),

        );

      }

      setError(error.message || 'Failed to submit proposal. Please try again.');

      setShowError(true);

    } finally {

      setIsSubmitting(false);

    }

  };

  const scrollUp = () =&gt; {

    smoothScrollToElement(scrollTargetRef);

  };

  const handleNextTab = () =&gt; {

    if (activeTab === 'details') {

      const capturedMarkdown = {

        description: descriptionEditorRef.current?.getMarkdown() || '',

        impactToEcosystem:

          impactOnEcosystemEditorRef.current?.getMarkdown() || '',

        objectives: objectivesEditorRef.current?.getMarkdown() || '',

        milestones: milestonesEditorRef.current?.getMarkdown() || '',

        impact: impactEditorRef.current?.getMarkdown() || '',

        budgetBreakdown: budgetBreakdownEditorRef.current?.getMarkdown() || '',

      };

      setMarkdownData(capturedMarkdown);

    }

    const currentIndex = tabs.findIndex((tab) =&gt; [tab.id](http://tab.id) === activeTab);

    if (currentIndex &lt; tabs.length - 1) {

      setActiveTab(tabs\[currentIndex + 1\].id);

      scrollUp();

    }

  };

  const handlePreviousTab = () =&gt; {

    const currentIndex = tabs.findIndex((tab) =&gt; [tab.id](http://tab.id) === activeTab);

    if (currentIndex &gt; 0) {

      setActiveTab(tabs\[currentIndex - 1\].id);

      scrollUp();

    }

  };

  return (

    &lt;div className="bg-background min-h-screen"&gt;

      &lt;div className="container mx-auto max-w-4xl px-4 py-8"&gt;

        &lt;div ref={scrollTargetRef} className="mb-8 text-center"&gt;

          &lt;Title level="5" className="text-foreground"&gt;

            Submit a proposal

          &lt;/Title&gt;

        &lt;/div&gt;

        &lt;div className="mb-8"&gt;

          &lt;div className="border-border w-full border-b"&gt;

            &lt;TopNav

              tabs={tabs}

              activeTabId={activeTab}

              onTabChange={setActiveTab}

            /&gt;

          &lt;/div&gt;

        &lt;/div&gt;

        &lt;div className="border-border bg-card rounded-lg border p-6 shadow-sm"&gt;

          &lt;div className="mb-8"&gt;

            {activeTab === 'details' && (

              &lt;DetailsTab

                formData={formData}

                handleInputChange={handleInputChange}

                descriptionEditorRef={descriptionEditorRef}

              /&gt;

            )}

            {activeTab === 'funds' && (

              &lt;FundsTab

                formData={formData}

                handleInputChange={handleInputChange}

              /&gt;

            )}

            {activeTab === 'review' && &lt;ReviewTab formData={formData} /&gt;}

          &lt;/div&gt;

          {activeTab === 'details' ? (

            &lt;div className="pt-6"&gt;

              &lt;Button

                variant="primary"

                onClick={handleNextTab}

                className="w-full"

              &gt;

                Next

              &lt;/Button&gt;

            &lt;/div&gt;

          ) : (

            &lt;div className="flex items-center justify-between gap-4 pt-6"&gt;

              {activeTab !== 'details' && (

                &lt;div className="text-primary-base w-1/4"&gt;

                  &lt;Button

                    variant="outline"

                    onClick={handlePreviousTab}

                    className="w-full"

                  &gt;

                    Back

                  &lt;/Button&gt;

                &lt;/div&gt;

              )}

              &lt;div className={activeTab === 'details' ? 'w-full' : 'w-3/4'}&gt;

                {activeTab !== 'review' ? (

                  &lt;Button

                    variant="primary"

                    onClick={handleNextTab}

                    className="w-full"

                  &gt;

                    Next

                  &lt;/Button&gt;

                ) : (

                  &lt;Button

                    variant="primary"

                    onClick={handleSubmit}

                    disabled={isSubmitting}

                    className="w-full"

                  &gt;

                    {isSubmitting ? 'Submitting...' : 'Submit proposal'}

                  &lt;/Button&gt;

                )}

              &lt;/div&gt;

            &lt;/div&gt;

          )}

        &lt;/div&gt;

      &lt;/div&gt;

      {/\* Success Modal - shown after transaction confirmation \*/}

      &lt;Modal

        isOpen={showConfirmation}

        onClose={() =&gt; setShowConfirmation(false)}

        title="Proposal Confirmed!"

        description="Your proposal has been confirmed on the blockchain"

        size="lg"

        actions={\[

          {

            label: 'View My Submissions',

            variant: 'primary',

            onClick: () =&gt; {

              setShowConfirmation(false);

              window.location.href = [routes.my](http://routes.my).submissions;

            },

          },

          {

            label: 'Close',

            variant: 'outline',

            onClick: () =&gt; setShowConfirmation(false),

          },

        \]}

      &gt;

        &lt;div className="py-4 text-center"&gt;

          &lt;div className="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-full bg-green-100"&gt;

            &lt;svg

              className="h-6 w-6 text-green-600"

              fill="none"

              stroke="currentColor"

              viewBox="0 0 24 24"

            &gt;

              &lt;path

                strokeLinecap="round"

                strokeLinejoin="round"

                strokeWidth={2}

                d="M5 13l4 4L19 7"

              /&gt;

            &lt;/svg&gt;

          &lt;/div&gt;

          &lt;p className="text-foreground mb-4"&gt;

            Your proposal "{formData.title}" has been successfully confirmed on

            the blockchain and is now pending review.

          &lt;/p&gt;

        &lt;/div&gt;

      &lt;/Modal&gt;

      {/\* Error Modal \*/}

      &lt;Modal

        isOpen={showError}

        onClose={() =&gt; setShowError(false)}

        title="Submission Error"

        description="There was an issue submitting your proposal"

        actions={\[

          {

            label: 'Try Again',

            variant: 'primary',

            onClick: () =&gt; setShowError(false),

          },

        \]}

      &gt;

        &lt;div className="py-4 text-center"&gt;

          &lt;p className="text-foreground"&gt;{error}&lt;/p&gt;

        &lt;/div&gt;

      &lt;/Modal&gt;

      {/\* Loading Overlay \*/}

      &lt;Modal

        isOpen={isSubmitting}

        onClose={() =&gt; {}}

        title="Submitting Proposal"

        description=" Please hold on as we do some magic"

        actions={\[\]}

      &gt;

        &lt;CardanoLoaderSVG size={64} /&gt;

      &lt;/Modal&gt;

    &lt;/div&gt;

  );

}